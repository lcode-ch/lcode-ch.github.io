---
layout: post
title: Spark入门：05-Spark核心编码
category: spark
tags: [spark]
excerpt: Spark核心编码
lock: need
---

Spark计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是：

- RDD : 弹性分布式数据集

- 累加器：分布式共享只写变量

- 广播变量：分布式共享只读变量

接下来我们一起看看这三大数据结构是如何在数据处理中使用的。

## 5.1 RDD

### 5.1.1 什么是RDD

```sql
-- 什么是RDD?
弹性分布式数据集，一种数据处理的模型&数据结构，是一个抽象类。如汽车模型，航母模型、手机模型等。

-- RDD的特点：
	1. 可分区：提高消费能力，更适合并发计算，类似kafka的消费者消费数据
	2. 弹性：变化，可变。
		a、存储弹性：可以在磁盘和内存之间自动切换；
		b、容错弹性：数据丢失可以自动回复；
		c、计算弹性：计算出错后重试机制；
		d、分区弹性： 根据计算结果动态改变分区的数量。
     3. 不可变：类似不可变集合
          RDD只存储计算的逻辑，不存储数据，计算的逻辑是不可变的，一旦改变，则会创建新的RDD；
     4. RDD ：一个抽象类，需要子类具体实现,说明有很多种数据处理方式
```

## 5.2 IO

```sql
-- IO流分为：
          字节流          字符流
输入流   inputStream       read
输出流   outPutStream      write

节点流  File+ 字符流/字节流
处理流  buffer(  File+ 字符流/字节流 )

-- 解读如下三张图流程
图1：使用字节流读取一个文件的内容并打印到控制台，使用一个文件节点流，只能读取一部分数据，然后打印，然后再读取一部分数据，再进行打印，慢；
图2：增加一个缓冲流，将获取的数据暂时先存放在内存的一个缓冲区内，等到一定的数据量以后，再统一处理；
图3：发现字节流获取的数据，打印到控制台，我们是不认识的，中间使用一个字节流转字符流，将读取的数据转化为字符流，然后再将字符缓存到内存，待达到一定的数据量以后，再往控制台上打印。

综上，发现，如上的过程属于装饰者模式，前者的结果传递到后者，一层一层的包裹起来。
```

![image-20200604221413351](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221413.png)
![image-20200604221320626](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221320.png)

![image-20200604221352619](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221352.png)

## 5.3 RDD执行原理

```sql
-- RDD的执行原理：
1. 类似IO处理；
2. 体现装饰者模式，通过new的方式体现装饰者模式；
3. 延迟加载，RDD只是封装了逻辑，只要当执行算子（如collect()）执行时，才会开始执行。

-- 与IO的区别：
RDD不保存数据，只保留逻辑，但是IO会保存数据

-- 如何理解RDD
不可变集合 --> 增加新的数据 --> 创建新的集合
RDD --> 扩展新的功能 --> 创建新的RDD
```

![image-20200604221511071](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221511.png)

```sql
-- 解读RDD：
1.一个执行器Executor可以有多个core核，默认情况下一个分区产生一个task，一个 task可以被一个core执行，由于一个执行器可以有多个核，所以一个执行器可以执行多个task；
```

![image-20200604221612259](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221612.png)

## 5.4 RDD的核心属性

### 5.4.1 分区列表

RDD数据结构中存在分区列表，用于执行任务时并行计算，是实现分布式计算的重要属性。

```sql
-- 什么是分区列表
就是RDD中的多个分区
```

![image-20200604225854664](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604225854.png)

### 5.4.2 分区计算函数

```sql
-- 什么是分区计算函数
Spark在计算时，是使用分区函数对每一个分区进行计算
```

![image-20200604225648602](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604225648.png)

### 5.4.3 RDD之间的依赖关系

RDD是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个RDD建立依赖关系。

![image-20200604225800723](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604225800.png)

### 5.4.4 分区器（可选）

Spark在计算时，是使用分区函数对每一个分区进行计算

```sql
-- 什么是分区器
数据进入分区的规则，对数据进行分区。
1. 只能是KV键值对的数据可以进行分区
2. 默认没有分区器
```

![image-20200604230017251](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604230017.png)

### 5.4.5 首选位置（可选）

```
计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算
```

![image-20200604230048390](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604230048.png)

![20200603003308](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604230102.png)

## 5.5 基础编程

### 5.5.1 创建RDD

```sql
-- 4种创建RDD的方式：
1. 从内存(集合)中创建
2. 从磁盘中创建
3. 从其他RDD中创建：RDD调用新的逻辑就是创建新的RDD
4. 直接创建：通过new的方式，spark框架内部会使用
```

#### 5.5.1.1 从内存(集合)中创建

```sql
-- 方法1：parallelize(形参)
    1. 方法：使用parallelize(形参)：创建一个RDD
    2. 形参：有两个参数：
       "参数1"：seq:Seq[T]，带泛型的序列，可以传递一个List集合
       "参数2"：numSlices：int = defaultParallelism，"后面讲平行度和分区时详细讲"。

 -- 方法2：makeRDD(形参)
      1.发现：底层还是调用的parallelism方法，所以参数和其一模一样，只是方法名更好理解
      	parallelize(seq, numSlices)
```

```scala
//构建Spark的环境和创建和Spark的连接说明
    /*
    setMaster(master：String)：指明Spark运行的环境
    形参：local[]    学习期间暂时使用本地环境
    local[1]:代表单核
    local[4]:代表4核
    local[*]:代表最大核数，在设备管理器中可以查看，我的电脑是12核

    SetAppName(name:String):执行程序的名称。
     */

    val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("wordCount")
    val sc = new SparkContext(sparkConf)
```

```scala
 val list = List(1, 2, 3, 4)

//方法1：parallelize(list)
    val datas: RDD[Int] = sc.parallelize(list) 
    datas.saveAsTextFile("output")//local,单核，数据只存在一个分区
    datas.saveAsTextFile("output1")//local[3],三核，数据存在3个分区
    datas.saveAsTextFile("output2")
	//local[*],最大核数：12核，生成12个分区，但是由于list数据没有那么多，所以有些分区没有数据

//方法2：makeRDD(list)
    val rdd: RDD[Int] = sc.makeRDD(list)
//将RDD的处理后的数据保存到分区文件中
    rdd.saveAsTextFile("output3")
```

#### 5.5.1.2 从外部(Disk)存储中创建RDD

```sql
-- 从本地磁盘中创建RDD
    1. 方法：textFile(形参)
    2. 形参：有两个参数
       "参数1"：path:String，表示文件的路径
              "表示方式"：
              a、可以表示一个文件
              b、可以表示一个文件夹
              c、还可以使用通配符"星号"的方式表示多个文件
              如val rdd: RDD[String] = sc.textFile("input/*.txt")
              "路径说明"：
              a、可以是相对路径，在IDEA中，从当前项目的根目录下往下找，path路径根据环境的不同自动发生改变
              b、也可以是绝对路径
              c、还可以指向第三方存储路径，如HDFS
      "参数2"：minPartitions：Int = defaultMinPartitions
              指建议产生的RDD的最小分区数，后面与并行度重点展开。
          	  suggested minimum number of partitions for the resulting RDD
          	  
 -- 说明：spark读取文件时，默认是采用hadoop读取文件的规则，按行读取。
```

```scala
   //环境准备和连接Spark
    val sparkConf: SparkConf = new SparkConf().setMaster("local").setAppName("wordCount")
    val sc = new SparkContext(sparkConf)

    //创建RDD
    val rdd: RDD[String] = sc.textFile("input")
   //将RDD的处理后的数据保存到分区文件中
    rdd.saveAsTextFile("output")
```

### 5.5.2 并行度与分区

```sql
-- 理解一下什么是并行度和分区
   并行度：parallelism，指整个集群同时执行任务的数量
   分区：partitions，数据在RDD中分区，在RDD模型中，一个分区将生成一个task，且task执行互不影响。

-- 并行度和分区的关系
   默认情况下（即资源充足的情况下），一个分区生成的一个task，一个task为一个并行度。
   也就是：分区数量 = 并行度。
   
-- 说明：并行度还和集群的总核数有关，所以资源充足就是指集群可用的核数  core >= task数量，如果可用的core < task数量(即分区数量)，那么并行度就比task小。
```

#### 5.5.2.1 从内存中读取数据

```sql
1. 创建RDD的方式为：
-- 方法1：parallelize(形参)
-- 方法2：makeRDD(形参)

2. 形参为：( seq:Seq[T],numSlices：int = defaultParallelism)
   形参2：numSlices：int = defaultParallelism
   "参数的含义"：表示集合被切分分区数量，有默认值： number of partitions to divide the collection into。
       分区数量：
       a、如果传参数了，那么按照传递的参数为分区数量；
       b、如果没有传参数，则使用默认值，默认值的情况如下：
          源码中：
          默认值：defaultParallelism，"默认并行度"，调用下面这个函数：
          scheduler.conf.getInt("spark.default.parallelism", totalCores)
           ①如果连接spark的配置中设定spark.default.parallelism参数了，那么就等于设定的参数
           ②如果没有设置，那么就等于机器总核数
               "什么是机器总核数?"
               机器总核数 = 当前环境中可用核数
               local => 单核（单线程）=> 1
               local[4] => 4核（4个线程） => 4
               local[*] => 最大核数 => 我的电脑最大核数为12
                
3. 通过参数2知道了数据的分区数量，那么数据进入不同分区的规则是什么？
   如下有三个案例，通过设置不同的分区数量，确认数据在分区中的情况。
   
   --规则总结：
   内存中的数据在分区中基本上是平均分配的。
   如果：数据条数 % 分区数 == 0   --> 平均分配，如有4个数据，2个分区，则前面两个进入分区0，后面两个进入分区1
         数据条数 % 分区数 !=0   --> 会采用一种基本算法实现分配，源码中具体的实现方法如下
   
   
       -- 计算每个分区中数据的起止索引
     def positions(length: Long, numSlices: Int): Iterator[(Int, Int)] = {
      (0 until numSlices).iterator.map { i =>
        val start = ((i * length) / numSlices).toInt
        val end = (((i + 1) * length) / numSlices).toInt
        (start, end)
      }
    }
        -- Seq表示内存中的集合
        seq match {
        case _ =>
        val array = seq.toArray  -- 将集合中的元素转换为数组
        -- 调用了上面的计算分区索引的方法
        positions(array.length, numSlices).map { case (start, end) =>
            array.slice(start, end).toSeq  -- 根据索引位置对数据进行切分，确定哪些数据进入哪个分区 
        }.toSeq
```



```scala
// 情况1：从内存中读取数据时，集群的并行度、分区及数据进入分区的规则
    //创建spark环境和连接spark
    val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("wordcount")
    val sc = new SparkContext(sparkConf)

    //准备数据
    val list = List(1,2,3,4)

    /*
    情况1：设定分区为2。
    解析结果文件：
    1. 生成两个分区文件
    2. 分区数据如下：
       分区0：1 2
       分区1：3 4

     */
 val rdd1: RDD[Int] = sc.makeRDD(list,2)
 rdd1.saveAsTextFile("output1")

    /*
   情况2：设定分区为4。
   解析结果文件：
   1. 生成4个分区文件
   2. 分区数据如下：
      分区0：1
      分区1：2
      分区2：3
      分区3：4

    */
  val rdd2: RDD[Int] = sc.makeRDD(list,4)
  rdd1.saveAsTextFile("output2")

    /*
      情况3：设定分区为3。
      解析结果文件：
      1. 生成3个分区文件
      2. 分区数据如下：
         分区0：1
         分区1：2
         分区2：3 4       
       */
    val rdd3: RDD[Int] = sc.makeRDD(list,3)
    rdd1.saveAsTextFile("output3")

```

#### 5.5.2.2 从文件中读取数据

1. 单文件读取情况

```sql
-- 问题1：分区数如何确定：
   
   方法解读：textFile(path:String,minPartitions：Int = defaultMinPartitions)
   
   --1.建议最小分区数：
    参数2：minPartitions：Int = defaultMinPartitions，指建议产生的RDD的最小分区数

    --2. 情况一：假如使用默认值：
        默认值：defaultMinPartitions，源码中调用了方法：取defaultParallelism【默认并行度】和2的最小值
        源码：def defaultMinPartitions: Int = math.min(defaultParallelism, 2)
         defaultParallelism：源码中调用：
         scheduler.conf.getInt("spark.default.parallelism", totalCores)
         ①如果连接spark的配置中设定spark.default.parallelism参数了，那么就等于设定的参数
         ②如果没有设置，那么就等于总核数
               "什么是总核数?"
               机器总核数 = 当前环境中可用核数
                local => 单核（单线程）=> 1
                local[4] => 4核（4个线程） => 4
                local[*] => 最大核数 => 我的电脑最大核数为12

     --3. 情况二：当传递了参数以后：使用传递的参数值。

     --4. 什么是最小分区数？
        所谓的最小分区数，取决于总的字节数是否能整除分区数，并且看剩下的字节数/每个分区的字节数是否
        大于10%，如果大于10%，则剩余的字节数会生产一个新的分区。

    --5. 实际上的分区数是多少呢？
    1. 实际的分区数 >= 设定的RDD最小分区数
    2. 算法：文件的字节数 / RDD设定的最小分区数  = result
       a、如果恰好整除，则实际的分区数=设定的RDD最小分区数
       b、如果除不尽，有余数，如果余数 / result <= 10% ,则实际的分区数=设定的RDD最小分区数
                            如果余数 / result > 10% ,则实际的分区数= 设定的RDD最小分区数 + 1
============================================================================================================
--问题2：数据依据什么规则进入不同的分区？

    通过查看源码发现，Spark读取文件采用的是hadoop的读取规则。
    1、切片规则："以字节的方式来切分数据"
    2、数据读取规则："按行读取"。

    回答问题2之前，我们先来回答如下两个问题。
    问题a：文件到底切成几片(也就是分区的数量)
          按照文件的字节数，确定预计的切片数量。

    问题b：分区数据是如何进行存储的？
        1.换行符为2个字节
        2.分区数据按行为单位进行读取，一行的数据不会被拆分。

    "规则"：数据进行不同分区的规则有二，二者合并一起使用。
    规则一：数据起始偏移量和字节数。
    规则二：数据偏移量(offset)

    --具体是什么意思？一起跟着下面案例来详细认识。
```

- 案例1：

```sql
举例1：
    --word1数据：
        12@@
        234
        说明：换行符(用@@代替)为2个字节，Spark是按照行进行读取数据。

    -- 第一步：建议生成RDD的最小分区数： => 2
    -- 第二步：计算实际的分区数：=> 3
               1.计算文件的字节数：=> 7个字节
               2.计算整除的结果和余数： 7/2=3 ..1
               3.计算余数和每个分区字节数的比率：1/3 > 10% => 生成一个新的分区  => 3
    -- 第三步：计算每个分区的数据起始偏移量和每个分区的字节数
               分区0：(起始偏移量，字节数) =>(0,3) =>(0,3)
               分区1：(起始偏移量，字节数) =>(3,3) =>(3,6)
               分区2：(起始偏移量，字节数) =>(6,1) =>(6,7)
    -- 第四步：计算每行数据的偏移量
               12@@    => 0 1 2 3
               234     => 4 5 6
    -- 第五步：数据的分配：按行读取，数据只会被读取一次
               分区0 => 读取索引为0/1/2/3的数据，读取12
               分区1 => 读取索引为3/4/5/6的数据，发现3已经被读取了，所以读取4/5/6索引的数据，读取：234
               分区2 => 读取索引为6/7的数据,发现6已经被读取，索引7无数据，所以分区2没有数据。
     */
     
	  	val rdd1: RDD[String] = sc.textFile("input/word1",2)
  		rdd1.saveAsTextFile("output")
```

- 案例2

```sql
 举例2：
    --word2数据：
        1@@
        2@@
        3@@
        4
        说明：换行符(用@@代替)为2个字节，Spark是按照行进行读取数据。

    -- 第一步：建议生成RDD的最小分区数： => 3
    -- 第二步：计算实际的分区数：=> 4
               1.计算文件的字节数：=> 10个字节
               2.计算整除的结果和余数： 10/3=3 ..1
               3.计算余数和每个分区字节数的比率：1/3 > 10% => 生成一个新的分区
    -- 第三步：计算每个分区的数据起始偏移量和每个分区的字节数
               分区0：(起始偏移量，字节数)=(0,3) =>(0,3)
               分区1：(起始偏移量，字节数)=(3,3) =>(3,6)
               分区2：(起始偏移量，字节数)=(6,3) =>(6,9)
               分区3：(起始偏移量，字节数)=(9,1) =>(9,10)
    -- 第四步：计算数据的偏移量
               1@@  => 0 1 2
               2@@  => 3 4 5
               3@@  => 6 7 8
               4    => 9
    -- 第五步：数据的分配：按行读取，数据只会被读取一次
               分区0 => 读取索引为0/1/2/3的数据，因为第一行数据不够，所以第二行也被读取，
                       所以实际读取了索引为0/1/2/3/4/5的数据，读取1 2
               分区1 => 读取索引为3/4/5/6的数据，发现3/4/5已经被读取了，所以只能读取索引为6的数据，
                        所以读取6索引所在行，最后读取了索引为6/7/8的数据，读取：3
               分区2 => 读取索引为6/7/8/9的数据,发现6/7/8已经被读取，所以只能读取索引为9的数据，
                        所以读取索引9所在行，最后读取了索引为9的数据，读取：4。
               分区3 =>  读取索引为9/10的数据,发现9已经被读取，索引10无数据，所以分区2没有数据。
    

    val rdd2: RDD[String] = sc.textFile("input/word2",3)
    rdd2.saveAsTextFile("output")
```

2. 多文件读取情况

```
多文件和单文件异同：
    1) 字节数为递归计算所有文件的字节数总和
    2）不能跨文件读取数据
    3）依然是按行读取
    4）计算每个分区的字节数的算法保持不变，
       但是总分区数增加的个数依据每个文件的字节数是否能整除每个分区的字节数而定。
```

- 案例解析

```scala
object Spark_FliePartitions {
  def main(args: Array[String]): Unit = {

    val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("FileParallelism")
    val sc = new SparkContext(sparkConf)

    /*
    文件1：
    12@@  => 0,1,2,3
    234   => 4,5,6
    文件2：
    1@@   => 0,1,2
    2@@   => 3,4,5
    3@@   => 6,7,8
    4     => 9
    计算过程：
    1.字节总数：7+10=17
    2.每个分区字节数：17/3=5 .. 2
    3.文件1：
       分区0：(0,5) =>12 234
       分区1：(5,7) =>空
     文件2：
       分区0：(0,5)  =>1 2
       分区1：(5,10) => 3 4
     */
    val rdd: RDD[String] = sc.textFile("input",3)
    rdd.saveAsTextFile("output")
    sc.stop()

  }

}

```

#### 5.5.3 RDD算子

```sql
RDD转换算子：
    -- 1. 什么是算子？
       认知心理学，解决问题的思路，也就是方法。
    -- 2. 所谓的RDD算子，其实就是将旧的RDD通过方法的调用转换为新的RDD
    -- 3. 既然算子也是方法，那么为什么叫做算子呢？是因为RDD的方法有别于其他对象的方法，后面会详细讲到怎么个不相同法。
    -- 4. 算子的分类：
      根据RDD处理数据的方式不同分为：value类型、双value类型、key-value类型。
```

##### 5.5.3.1 map

```sql
--算子：map(形参)：
    1. 作用：将处理的数据逐条进行映射处理，"类比scala中的map，对数据进行结构转换"
    2. 形参：def map[U: ClassTag](f: T => U): RDD[U]
    3. 基本使用如下：
```

```scala
    val list = List(1,2,3,4)
    val rdd: RDD[Int] = sc.makeRDD(list)
    val rddmap: RDD[Int] = rdd.map(_*2)
    println(rddmap.collect().mkString(","))
```

```sql
-- 关于map算子的两个问题

    --问题1：分区的问题：RDD有分区列表，每个RDD都有相同的分区计算函数，那么新的RDD与旧的RDD的分区关系是什么？
          默认分区的数量保持不变，数据会转换后输出。
   
   --问题2：Map中数据处理的顺序是怎么样的？
          通过如下验证发现：
          a、分区内数据按照顺序依次执行，且第一条数据的所有逻辑执行完成以后再执行第二条数据，依次类推
          b、分区间的数据执行是没有顺序，而且无需等待，即分区间执行逻辑互不影响，各自执行各自的逻辑。
```

- 验证如下：

```scala
//测试：新旧RDD分区的关系 
   val rdd1: RDD[Int] = sc.makeRDD(list,2)
   val rddmap1: RDD[Int] = rdd1.map( num => num * 2})
  //将数据输出到本地文件中，查看分区数量及分区内的数据
   rdd1.saveAsTextFile("output1")
   rddmap1.saveAsTextFile("output")

//测试：分区间的执行顺序
    val rddmap2: RDD[Int] = rdd1.map( num => {println("mapA ->" + num );num * 2})  
    val rddmap3: RDD[Unit] = rddmap1.map(num => println("mapB ->" + num))
	//collect方法不会转换RDD，会触发作业的执行,所以将collectt这样的方法称之为行动（action）算子	
	rddmap3.collect()
```

- 练习：

```scala
    //练习：从服务器日志数据apache.log中获取用户请求URL资源路径
    val rdd: RDD[String] = sc.textFile("input/apache.log")
    val result: Array[String] = rdd.map(
      str => {
          //按照空格拆分一条数据
        val array: Array[String] = str.split(" ")
          //只取URL资源数据
        array(6)
      }
    ).collect()
   //遍历结果集
    result.foreach(println)

```

##### 5.5.3.2 mpaPartitions

```sql
--map()算子问题：
  在分区内只能每次获取一个一个的数据，而且只有当前一个数据的所有逻辑执行完成以后才会执行下一个数据，这样一来，效率就相对比较慢。
  
--引出了另外一个算子:mpaPartitions(形参)

     --1. 形参：(f: Iterator[T] => Iterator[U],preservesPartitioning: Boolean = false): RDD[U]
           形参1：f: Iterator[T] => Iterator[U]，是一个函数
                 函数的形参：一个迭代器，内容为一个分区中所有的数据；
                 函数的返回：分区内每个数据经过转换以后数据形成的"迭代器"。
           参数2：暂时不管。
    --2. 返回结果：返回一个新的RDD
    --3. 算子的作用：
           将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行"任意的处理"，哪怕是过滤数据   
    --4. 与map()算子的不同点：
           map 算子是一个全量数据处理，不能丢失数据；
           mapPartitions 算子一次获取分区中所有的数据，那么可以执行迭代器所有的操作，如可以进行数据的过滤。
   --5. mapPartitions算子存在的问题
          如果一个分区的数据没有处理完，那么该分区内所有的数据都不会释放，即使是前面已经处理完的数据也不会释放
          容易出现内存溢出。 
   --6. map和mapPartitions()算子的选择：
      	  如果内存空间足够大，为了提高效率时，推荐使用mapPartitions()算子
```

- 代码演示

```scala
    val list = List(1,2,3,4)
    val rdd: RDD[Int] = sc.makeRDD(list,2)
    val rdd1: RDD[Int] = rdd.mapPartitions(iter => {
      //只要分区内为偶数的数据
      iter.filter(_% 2 ==0)

    })
    println(rdd1.collect().mkString(","))
```

- 练习

```scala
    //练习：获取每个数据分区的最大值
    val list = List(1,5,6,4,3,6)
    val rdd: RDD[Int] = sc.makeRDD(list,3)
    val result: RDD[Int] = rdd.mapPartitions(
      iter => {
        //求分区内的最大值，返回值为一个值，不是迭代器，所以使用List集合进行包装
        List(iter.max).iterator
      }
    )
    println(result.collect().mkString(","))
```

##### 5.5.3.3 mapPartitionsWithIndex

```sql
    1. 算子：mapPartitionsWithIndex (形参)
    
    2. 形参：(f: (Int, Iterator[T]) => Iterator[U],preservesPartitioning: Boolean = false)
       形参1：f: (Int, Iterator[T]) => Iterator[U]，是一个函数
                函数的形参：
                参数1：为分区号
                参数2：为一个迭代器，内容为一个分区中所有的数据；
                函数的返回：分区内每个数据经过转换以后数据形成的"迭代器"。
      形参2：暂时不管。
      
    3.算子的作用：将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，
                 哪怕是过滤数据，"在处理时同时可以获取当前分区索引"
```

- 代码演示

```scala
val list = List(1,2,3,4)
    val rdd: RDD[Int] = sc.makeRDD(list,2)
    //获取每个分区最大值以及分区号
    val rdd1: RDD[(Int, Int)] = rdd.mapPartitionsWithIndex(
      (num, iter) => {
        List((num, iter.max)).iterator
      })

    println(rdd1.collect().mkString(","))
```

- 练习

```scala
  //练习：获取第二个数据分区的数据    
    val list = List(1, 2, 3, 4, 5, 6)
    val rdd: RDD[Int] = sc.makeRDD(list, 3)
  
    val rdd1= rdd.mapPartitionsWithIndex(
      (num, iter) => {
        if (num == 1) {
          iter
        } else {
          Nil.iterator
        }
      })

    println(rdd1.collect().mkString(","))
```

##### 5.5.3.4 flatmap

```sql
     1. 算子：flatMap(形参)

     2. 形参：(f: T => TraversableOnce[U]):是一个函数
           函数的参数：分区内的一个一个的元素
           返回值：经过映射以后将将数据进行扁平化，返回一个可迭代的集合
     
     3. 作用：和scala中的作用完全一致，映射扁平
```

- 代码演示

```scala
    val list = List(List(1,2),List(3,4))
    val rdd: RDD[List[Int]] = sc.makeRDD(list)
    val rdd1: RDD[Int] = rdd.flatMap(list=>list)
    println(rdd1.collect().mkString(","))
```

- 练习

```scala
    //将List(List(1,2),3,List(4,5))进行扁平化操作
    val list = List(List(1, 2), 3, List(4, 5))
    val rdd: RDD[Any] = sc.makeRDD(list)
    val rdd1: RDD[Any] = rdd.flatMap(
      data => {
        data match {
          case list: List[_] => list
          case b => List(b)
        }
      }
    )
    println(rdd1.collect().mkString(","))
```

##### 5.5.3.5 glom

```sql
      1. 算子：glom(形参)
      2. 形参：空，无形参
      3. 返回值：RDD[Array[T]]，返回一个一个的数组，数组的数据来自同一个分区
      4. 作用：将同一个分区内的数据转换成数组。
```

- 代码实现

```scala
    val list = List(1, 2, 5, 6, 4, 3)
    val rdd: RDD[Int] = sc.makeRDD(list, 3)
    val rdd1: RDD[Array[Int]] = rdd.glom()
    rdd1.collect().foreach(array=>{println(array.mkString(","))})
```

- 练习

```scala
 //练习：计算所有分区最大值求和（分区内取最大值，分区间最大值求和）
    val list = List(1, 10, 8, 6, 2, 3)
    val rdd: RDD[Int] = sc.makeRDD(list, 3)
    //方法1：以分区来单位，进行处理
    val sum: Double = rdd.mapPartitions(
      iter => {
        List(iter.max).iterator
      }
    ).sum()
    println(sum)
    
    //方法2：使用glom
    println(rdd.glom().flatMap(array => List(array.max).iterator).sum())
```

##### 5.5.3.6 groupBy

```sql
     --1. 算子：groupBy(形参)
     --2. 形参：def groupBy[K](f: T => K,p:Partitioner)(implicit kt: ClassTag[K]): RDD[(K, Iterable[T])]
            形参1：f: T => K：是一个函数
                  函数的形参为：分区中的一个一个的元素
                  返回值为：返回分组的K
            形参2：p:Partitioner,指设定下游的分区数量，如果不设置，则默认为旧RDD的分区数量
            算子的返回值：返回一个元组
                  元组的第一个元素：表示分组的Key
                  元组的第二个元素：表示相同的key形成可迭代的集合
     --3. 作用：将数据根据指定的规则进行分组。
     --4. 特点：
            a、分区默认不变
            b、不同分区的数据会被重新打乱进入到不同的分区中；
            c、我们将上游的分区数据打乱重新组合到下游的分区中，这个操作称之为shuffle
            d、极限情况下，所有的数据会被分到一个分区
            e、一个组的数据在一个分区，但是并不是说一个分区中只有一个组
     --5. 存在的问题：    
       	    groupby方法会导致数据重新组合以后不均匀
     --6. 解决方案：
            通过传递参数，改变下游分区的数量。
```

- 代码

```scala
    /*
     1.一个组的数据在一个分区，但是并不是说一个分区中只有一个组
     奇偶分组，将数据分成两个组，结果文件中只有一个分区文件，分区文件中有两个分组。
      */

    val list = List(1,2,3,4,5,6,7,8)
    val rdd: RDD[Int] = sc.makeRDD(list,1)
    val rdd1: RDD[(Int, Iterable[Int])] = rdd.groupBy(_ % 2)
    rdd1.saveAsTextFile("output")

    /*
    2.当前有4个分区，奇偶分组只会有两个分组，所以结果文件中有4个分区文件，但是有两个分区分件中没有数据
     */

    val rdd: RDD[Int] = sc.makeRDD(list,4)
    val rdd1: RDD[(Int, Iterable[Int])] = rdd.groupBy(_ % 2)
    rdd1.saveAsTextFile("output")


    /*
    3.通过设置下游的分区数量解决分区无数据的情况,此时生成的结果文件只有两个分区
     */
        val rdd: RDD[Int] = sc.makeRDD(list,4)
        val rdd1: RDD[(Int, Iterable[Int])] = rdd.groupBy(((num :Int) => num % 2),1)
        rdd1.saveAsTextFile("output")
```

- 练习

```scala
//    小功能：将List("Hello", "hive", "hbase", "Hadoop")根据单词首写字母进行分组。
    val list = List("Hello", "hive", "hbase", "Hadoop")
    val rdd: RDD[String] = sc.makeRDD(list)
    val rdd1: RDD[(String, Iterable[String])] = rdd.groupBy(word => word.substring(0,1))
    println(rdd1.collect().mkString(","))

//    小功能：从服务器日志数据apache.log中获取每个时间段访问量。
    val rdd: RDD[String] = sc.textFile("input/apache.log")
    val rdd1: RDD[String] = rdd.flatMap(str => {
      val datas: ArrayOps.ofRef[String] = str.split(" ")
      List(datas(3).substring(11, 13))
    })
    val rdd2: RDD[(String, Iterable[String])] = rdd1.groupBy(time=>time)
    println(rdd2.flatMap(data => List((data._1, data._2.size))).sortBy(_._1).collect().mkString(","))

//    小功能：WordCount。
      val rdd: RDD[String] = sc.textFile("input/word1")
      val wordcount: String = rdd.flatMap(_.split(" ")).groupBy(word => word)
      .map(tuple => (tuple._1, tuple._2.size)).collect().mkString(",")
     println(wordcount)

```

##### 5.5.3.7 filter

```sql
    --1. 算子：Filter(形参)
    --2. 形参：(f: T => Boolean)：是一个函数，用法和scala中的fliter类似
            函数的形参：RDD中的一个一个的数据
            返回值：ture或者false
            true：表示数据被保留下来
            false：表示数据被过滤掉
    --3. 作用：将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。
    --4. 特点：
           a、分区不变
           b、分区内的数据可能不均衡，生产环境下，可能会导致数据倾斜
```

- 代码演示

```scala
    val list = List(1,2,3,4)
    val rdd: RDD[Int] = sc.makeRDD(list)
    val rdd1: RDD[Int] = rdd.filter(data => data % 2 ==0)
    println(rdd1.collect().mkString(","))      
```

- 练习

```scala
//练习：从服务器日志数据apache.log中获取2015年5月17日的请求路径
    val rdd: RDD[String] = sc.textFile("input/apache.log")
    val rdd1: RDD[(String, String)] = rdd.map(data =>(data.split(" ")(3),data.split(" ")(6)) )
    val rdd2: RDD[(String, String)] = rdd1.filter(tuple => {
      tuple._1.substring(0, 10) == "17/05/2015"
    })
    rdd2.collect().foreach(println)
```
