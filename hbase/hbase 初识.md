# hbase 初识

参考
[1][1]
[2][2]
[3][3]

[1]:https://www.ibm.com/developerworks/cn/analytics/library/ba-cn-bigdata-hbase/index.html
[2]:https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247485390&idx=1&sn=3d2ee4b5e0b94d8a88368e98aae24284&chksm=ebd746cfdca0cfd9ce76725501a4e6e384cd11aa2f2e769475d34c513382ad623df71548c68a&token=1134046556&lang=zh_CN&scene=21#wechat_redirect
[3]:https://juejin.im/post/5c666cc4f265da2da53eb714

## 行式存储与列式存储

### 行列对比

* 行式存储倾向于结构固定，列式存储倾向于结构弱化。
  * （行式存储相当于套餐，即使一个人来了也给你上八菜一汤，造成浪费；列式存储相等于自助餐，按需自取，人少了也不浪费）
* 行式存储一行数据只需一份主键，**列式存储一行数据需要多份主键**。
* 行式存储存的都是业务数据，列式存储除了业务数据外，还要存储列名。
* 行式存储更像一个Java Bean，所有字段都提前定义好，且不能改变；列式存储更像一个Map，不提前定义，随意往里添加key/value。
* **键由行关键字、列关键字和时间戳构成**

![Xnip2019-08-14_15-35-08](/assets/Xnip2019-08-14_15-35-08_s2yj3rxg9.png)

### 官方介绍

Apache Hbase是Hadoop数据库，一个分布式、可扩展、大数据存储。
当你需要随机地实时读写大数据时使用Hbase。它的目标是管理超级大表-数十亿行X数百万列。
Hbase是一个开源的、分布式的、**带版本的**、非关系型数据库，模仿谷歌的BigTable。BigTable使用Google File System作为分布式数据存储，**同理Hbase使用HDFS**。

### 基本概念

$1.行式存储与列式存储$

$2.数据模型$

>数据模型：Schema-->Table-->Column Family-->Column-->RowKey-->TimeStamp-->Value

$3.Column Family(列簇)$

$4.Column Qualifier(列修饰符)$

$5.Column(列)/列族:列修饰符$

在Hbase中一个列族（Column Family）和一个列修饰符（Column Qualifier）组合起来才叫一个列（Column），使用冒号（:）分割，列族:列修饰符

$6.row key(行键)$

在传统数据库中每一行的唯一标识符叫做主键，在Hbase中叫做row key（行键），HBASE 中行键相同的数据组成了原来数据表中的一行

$7.time stamp(时间戳)或版本号$

数据在进入Hbase时都会被打上一个时间戳，这个时间戳可以作为版本号来使用。
在t1时间我存入一个人的基本信息，之后发现姓名错了，在t2时间又更新了姓名，此时并不会去更新原来的那条数据，而是又插入了一条新数据且打上新的时间戳。
此时去查询获取的是新数据，仿佛是更新了，但其实只是默认返回了最新版本的数据而已

$8.cell(单元格)$

一个行键、列族、列修饰符、数据和时间戳组合起来叫做一个单元格（Cell）。这里的行键、列族、列修饰符和时间戳其实可以看作是定位属性（类似坐标），最终确定了一个数据。

$9.row(行)$

一个行键、一到多列（包括数据）组合起来叫做一行（Row）。下图中所有1001的数据合起来相当于Hbase中的一行，1002的相当于另一行：

$10.table(表)$

在Hbase中，只要确定了列族（具体的列不用管），表（Table）就确定了。

**`这里我们需要特别注意，每一个 Region 都只存储一个 列簇（Column Family） 的数据，并且是该 CF 中的一段（按 Row 的区间分成多个 Region）。而每个 Region 包含着多个 Store 对象。每个 Store 包含一个 MemStore，和一个或多个 HFile。所以同一个 Region 下的 Store，或者同一个 Store 下的 HFile 存储的数据都同属于一个列簇（Column Family）。`**

官方文档中提醒：把传统数据库中的表/行/列的概念用在Hbase中不是一个有帮助的类比。相反可以把Hbase的表想象成一个多（两）维Map（Map套Map）。列族是第一维，列修饰符是第二维。

![Xnip2019-08-14_15-38-23](/assets/Xnip2019-08-14_15-38-23.png)

### hbase 优化

复制或删除很多小文件时，非常耗时。如果把它们打包成一个压缩文件，再复制或删除，会快很多。

所以可以通过减少写入的文件个数来优化。如果我们只写一个文件，且每次都在文件末尾追加，这应该最大限度的减少了磁头的移动。

这种方式在Hbase中叫做预写日志，即Write Ahead Log（WAL）。所有的写入操作只要把数据追加到这个日志文件中就立即返回。

一个Region Server服务器只有一个这样的WAL文件，被服务器上的所有Region以及它里面的所有Store共享。

预写日志的数据格式并不适合进行最终的存储，所以在Store里面还有一个MemStore这样的数据结构驻留在内存里，会收集所有写入的数据，且按row key已排序。

当某些条件被满足时，MemStore中的数据会被flush到磁盘上进行持久化，最终这些数据以StoreFile的形式存储到HDFS中。

![Xnip2019-08-30_13-46-33](/assets/Xnip2019-08-30_13-46-33.png)

## HBASE优点

* 单项数据，直接选取，不需要全表扫描
* 同列簇数据会被相邻放置，即被存储在一起
* 查询效率高，支持高效率的实时查询
* hbase可以存储上百万列，又由于hbase基于hdfs来存储，所以hbase可以存储上亿行
  
在 HBASE 中列可以随时添加，但是列簇需在定义表的时候同时定义

>>注意：hbase中，其实所有列都是在列簇中，定义表的时候就需要指定列簇。生产环境由于性能考虑和数据均衡考虑，一般只会用一个列簇，最多两个列簇

## hbase 特点

* HBASE 只能通过 rowkey 来查询，当然可以通过部分消息中间件将 SQL翻译成 HBASE 的查询规则，从而也变相地支持了 SQL 查询

## Difference between Hive and HBase

* hive 是构建在 Hadoop 顶层，MapReduce 之上的 **`数据查询引擎`**（query engine），不支持实时查询，不支持行水平的 insert、update、delete；**`不是一个完整的数据库`**，操作 MapReduce，用作数据分析，高延迟。
* HBASE运行在 HDFS 之上的一个 **`分布式数据库`**，支持 nosql 数据库，支持实时查询，不受schema 约束，实时处理，低延迟。

具体详见[Difference between Hive and HBase][1]

[1]:https://www.geeksforgeeks.org/difference-between-hive-and-hbase/

![Xnip2019-08-15_11-31-42](/assets/Xnip2019-08-15_11-31-42.png)

$补充$

* hive是进行分析查询的，不支持实时查询，需要较长时间才能返回结果，通常用于离线分析，而 HBASE却非常适合用来进行大数据的实时查询。
* hive 一般只需要有 Hadoop 就可以工作了，而 HBASE 还需要借助 Zookeeper的协助
* hive 支持用 HQL（Hive定义了一种类似SQL的查询语言(HQL),将SQL转化为MapReduce任务在Hadoop上执行），而HBase 本身只提供了 Java 的 API 接口，并不直接支持 SQL 的语句查询，当然一些中间件可以将 SQL 转换成 HBASE 查询规则，从而也能支持SQL 查询
* 如果想要在 HBase 上使用 SQL，则需要联合使用 Apache Phonenix，或者联合使用 Hive 和 HBase。但是和上面提到的一样，如果集成使用 Hive 查询 HBase 的数据，则无法绕过 MapReduce，那么实时性还是有一定的损失。Phoenix 加 HBase 的组合则不经过 MapReduce 的框架，因此当使用 Phoneix 加 HBase 的组成，实时性上会优于 Hive 加 HBase 的组合，
* Hive 和 HBase 所使用的 **`存储层`**，默认情况下 Hive 和 HBase 的存储层都是 HDFS。但是 HBase 在一些特殊的情况下也可以直接使用本机的文件系统。例如 Ambari 中的 AMS 服务直接在本地文件系统上运行 HBase。

## HBASE 的存储逻辑

在 HBASE 中，**Row-key ➕CF ➕Qulifier(Column-key) ➕timestamp** 才可以定位到一个单元格数据（HBASE 中每个单元格默认有 3 个时间戳的版本数据）。在创建表的时候就需要制定表的 CF、Row-key以及 Qulifier。这里我们现年通过下图理解下 HBASE 中，逻辑上的数据排布和物理上的数据排布之间的关系。

![Xnip2019-08-15_15-37-52](/assets/Xnip2019-08-15_15-37-52.png)

从上图中我们看到 Row1 到 Row5 的数据分布在两个 CF 中，并且每个 CF 对应一个 HFile。并且逻辑上每一行中的一个单元格数据，对应于 HFile 中的一行，然后用户按照 Row-key 查询数据的时候，HBASE 会遍历两个HFile，通过相同的 Row-key 标识，将相关的单元格组织称行返回，这样便有了逻辑上的行数据。

🔑🔑🔑

用户以 Row-key 进行数据查询，依据定位三要素定位单元格数据，遍历多个 HFile格式文件（存储在 Store 中，一个 Region 由多个 Store 组成，一个 HRegion Server管理多个 Region）搜索出相同 Row-key 的数据组织成行后返回。

🔑🔑🔑

## HBase 与 RDBMS 的区别

首先让我们了解下什么是 ACID。ACID 是指数据库事务正确执行的四个基本要素的缩写，其包含：**原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）以及持久性（Durability）**。对于一个支持事务（Transaction）的数据库系统，必需要具有这四种特性，否则在事务过程（Transaction Processing）当中无法保证数据的正确性，交易过程极可能达不到交易方的要求。下面，我们就简单的介绍下这 4 个特性的含义。

* 原子性(Atomicity)是指一个事务要么全部执行,要么全部不执行。换句话说，一个事务不可能只执行了一半就停止了。比如一个事情分为两步完成才可以完成，那么这两步必须同时完成，要么一步也不执行，绝不会停留在某一个中间状态。如果事物执行过程中，发生错误，系统会将事物的状态回滚到最开始的状态。
* 一致性(Consistency)是指事务的运行并不改变数据库中数据的一致性。也就是说，无论并发事务有多少个，但是必须保证数据从一个一致性的状态转换到另一个一致性的状态。例如有 a、b 两个账户，分别都是 10。当 a 增加 5 时，b 也会随着改变，总值 20 是不会改变的。
* 隔离性（Isolation）是指两个以上的事务不会出现交错执行的状态。因为这样可能会导致数据不一致。如果有多个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，使得在同一时间仅有一个请求用于同一数据。
* 持久性(Durability)指事务执行成功以后,该事务对数据库所作的更改便是持久的保存在数据库之中，不会无缘无故的回滚。

在具体的介绍 HBase 之前，我们先简单对比下 HBase 与传统关系数据库的（RDBMS，全称为 Relational Database Management System）区别。如表 1 所示。

![Xnip2019-08-15_15-52-18](/assets/Xnip2019-08-15_15-52-18.png)

对于 RDBMS 来说，一般都是以 SQL 作为主要访问方式。而 HBASE是一种 “nosql” 数据库。HBASE 是大型分布式 “NoSql” 数据库。**从技术上来看，HBASE 更像是“数据存储”而非“数据库”（HBASE 和HDFS 都是大数据的存储层）**。因此，HBASE 缺少很多 RDBMS 特性，如列类型、二级索引，触发器和高级查询语言等。然而，HBASE 也具有许多其他特性并且支持线性化和模块化扩充。最明显的方式，我们可以通过增加 Region Server 的数量扩展 HBASE。并且 HBASE 可以放在普通的服务器中，例如将集群从 5 个扩充到 10 个 Region Server 时，存储空间和处理容量都可以同时翻倍，当然 RDBMS 也能很好地扩充，但仅对一个点来说，由其对一个单独数据库服务器而言，为了更好地性能，往往需要特殊的硬件和存储设备（往往价格也非常昂贵）。

## hbase 架构及工作流程

### HBASE 架构

前面我们提到过 HBASE 也是构建与 HDFS 之上，这是正确的，但是也不是完全正确。HBASE 其实也支持直接在本地文件系统之上运行，不过这样的 HBASE 只能运行在一台机器上，那么对于分布式大数据环境是没有意义的（这也是所谓的 HBASE 单机模式）。这里我们只要记得在分布式生产环境中，HBASE 需要运行在HDFS 之上，以 HDFS 作为其基础的存储设施。HBASE 上层提供访问数据的 Java API 层，供应用访问存储在 HBASE 上的数据。HBASE 集群主要有 master、Region server以及 Zookeeper 组成，具体模块如下图所示。

![Xnip2019-08-15_22-05-00](/assets/Xnip2019-08-15_22-05-00.png)

其中；

* **Master**

    HBASE master 用于协调多个 Region Server，侦测各个 Region server 之间的状态，通过调整和管理 Region 分布来实现 HRegion Server 的负载均衡。（其实当大量rowkey相近的数据都被分配到一个Region中，导致这个Region数据过大的时候，Region进行拆分，HMaster会对拆分后的Region重新分配RegionServer，这是HMaster的负载均衡策略。）HBASE 允许多个 master 节点共存，但是这需要 Zookeeper 的帮助。不过当多个 master 节点共存时，只有一个 master 是提供服务的，其他的 master 节点处于待命状态。当正在工作的 master 节点宕机时，其他 master 节点才会接管 HBASE 的集群。

* **Region Server**

    对于一个 Region Server 而言，其包括多个 Region。Region Server 的作用只是接管表格，以实现读写操作。Client 直接连接 Region Server，并通过通信获取 HBASE 的数据。对于 Region 而言，则是真正存放 HBASE 数据的地方，也就是说 Region 是 HBASE 可用性和分布式的基本单位。如果当一个表格很大，并由多个 CF 组成时，那么表的数据将会存储在多个 Region 之间，并在每个 Region 中会关联多个存储单元（Store）。

* **Zookeeper**

    对于 HBASE 而言，Zookeeper 的作用至关重要。首先，Zookeeper 是作为 HBASE Master 的 HA 解决方案。也就是说，是 Zookeeper 保证了至少有一个HBASE Master 处于运行状态，并且 Zookeeper 负责 Region 和 HRegion Server 的注册。其次，在 HBASE 中，每一张表都会有元信息，这些信息也是被存储为 HBASE 表（即也是用rowkey和value的键值存储），称为元信息表，也叫meta表，这是一种系统表，它存储在那个分布式协调服务Zookeeper 上面。所以，Client 要想访问 HBASE 上的数据时，需要先从 Zookeeper 中的 meta 表获取相关表的存储信息，然后才能到 Region 中去查询表数据。

一个完整的分布式 HBASE 的工作原理示意图如下：

![Xnip2019-08-15_22-05-42](/assets/Xnip2019-08-15_22-05-42.png)

### HBASE 工作流程

在上面的图中，我们需要注意一下下面这几个概念，即 Store、MemStore、StoreFile以及 HFile。带着这几个概念，我们完整地梳理一下 HBASE 的工作流程。

首先，我们需要知道 HBASE 的集群是通过 Zookeeper 来进行机器之间的协调的，也就是说 HBASE Master 和 Region Server 之间的关系是依赖 Zookeeper 来维护的。当一个 Client 需要访问 HBASE 集群时，Client 需要先和 Zookeeper 通信，然后才能找到对应的 Region Server。每个 Region Server 管理着多个 Region。对于 HBASE 来说，Region 是 HBASE 并行化的基本单元。因此数据也都存储在 Region 中。

**这里我们要特别注意，每一个 Region 都只存储一个 Column Family 的数据，并且是该 CF 中的一段（将所有 同一 CF 下的数据 按 Row-key 区间划分为多个 Region）**。Region 所能够存储的数据量是由上限的，当数据量达到上限（Threshold）时，Region进行拆分，HMaster会对拆分后的Region重新分配Region Server，拆分成多个 Region，数据的并行化处理性能也得到了提升，数据容量也得到提升。

每个 Region 包含着多个 Store 对象。**每个 Store 包含一个 MemStore，和一个或多个 HFile**。MemStore 便是数据在内存中的实体，并且一般都是**有序**的。当数据向 Region 写入的时候，会先写入 MemStore。当 MemStore 中的数据需要向底层文件系统倾倒（Dump）时（例如 MemStore 中的数据体积到达 MemStore 配置的最大值），Store 便会创建 StoreFile，而 StoreFile 就是对 HFile 一层封装。所以 MemStore 中的数据会最终写入到 HFile 中，也就是磁盘 IO。由于 HBase 底层依靠 HDFS，因此 HFile 都存储在 HDFS 之中。这便是整个 HBase 工作的原理简述。

我们了解了 HBase 大致的工作原理，那么在 HBase 工作过程中，如何保证数据的可靠性呢？带着这个问题，我们理解下 HLog 的作用。HBase 中的 HLog 机制是 **WAL（write ahead log 预写日志）** 的一种实现，而 WAL（一般翻译为预写日志）是事务机制中常见的一致性的实现方式。**每个 Region Server 中都会有一个 HLog 的实例，Region Server 会将更新操作（如 Put，Delete）先记录到 WAL（也就是 HLog）中，然后将其写入到 Store 的 MemStore，最终 MemStore 会将数据写入到持久化的 HFile 中（MemStore 到达配置的内存阀值）**。这样就保证了 HBase 的写的可靠性。如果没有 WAL，当 Region Server 宕掉的时候，MemStore 还没有写入到 HFile，或者 StoreFile 还没有保存，数据就会丢失。或许有的读者会担心 HFile 本身会不会丢失，这是由 HDFS 来保证的。在 HDFS 中的数据默认会有 3 份。因此这里并不考虑 HFile 本身的可靠性。

### HFile 结构

HFile 是 HBASE 持久化的存储文件，其结构如下图所示：

![Xnip2019-08-15_23-21-36](/assets/Xnip2019-08-15_23-21-36.png)

从图中我们可以看到 HFile 由很多个数据块（Block）组成，并且有一个固定的结尾块。其中的数据块是由一个 Header 和多个 Key-Value 的键值对组成。在结尾的数据块中包含了数据相关的索引信息，系统也是通过结尾的索引信息找到 HFile 中的数据。HFile 中的数据块大小默认为 64KB。**如果访问 HBase 数据库的场景多为有序的访问，那么建议将该值设置的大一些。如果场景多为随机访问，那么建议将该值设置的小一些。**一般情况下，通过调整该值可以提高 HBase 的性能。

如果要用很短的一句话总结 HBase，我们可以认为 HBase 就是一个有序的多维 Map，其中每一个 Row-key 映射了许多数据，这些数据存储在 CF 中的 Column。

## HBASE 的使用建议

### HBASE or RDBMS

之前我介绍了很多 HBase 与 RDBMS 的区别，以及一些优势的地方。那么什么时候最需要 HBase，或者说 HBase 是否可以替代原有的 RDBMS？对于这个问题，我们必须时刻谨记——HBase 并不适合所有问题，**其设计目标并不是替代 RDBMS，而是对 RDBMS 的一个重要补充，尤其是对大数据的场景**。当需要考量 HBase 作为一个备选项时，我们需要进行如下的调研工作。

* 首先，要确信有足够多数据，如果有上亿或上千亿行数据，HBase 才会是一个很好的备选。

* 其次，需要确信业务上可以不依赖 RDBMS 的额外特性，例如，列数据类型, 二级索引，SQL 查询语言等。

* 再而，需要确保有足够硬件。且不说 HBase，一般情况下当 HDFS 的集群小于 5 个数据节点时，也干不好什么事情 (HDFS 默认会将每一个 Block 数据备份 3 分)，还要加上一个 NameNode。

### HBase 的表格设计

首先，一个 HBase 数据库是否高效，很大程度会和 Row-Key 的设计有关。因此，如何设计 Row-key 是使用 HBase 时，一个非常重要的话题。随着数据访问方式的不同，Row-Key 的设计也会有所不同。不过概括起来的宗旨只有一个，那就是**尽可能选择一个 Row-Key**，可以使你的数据均匀的分布在集群中。这也很容易理解，因为 HBase 是一个分布式环境，Client 会访问不同 Region Server 获取数据。如果数据排布均匀在不同的多个节点，那么在批量的 Client 便可以从不同的 Region Server 上获取数据，而不是瓶颈在某一个节点，性能自然会有所提升。对于具体的建议我们一般有几条：

* 当客户端需要频繁的**写一张表**，**随机的 RowKey** 会获得更好的性能。
* 当客户端需要频繁的**读一张表**，**有序的 RowKey** 则会获得更好的性能。
* 对于时间连续的数据（例如 log），有序的 RowKey 会很方便查询一段时间的数据（Scan 操作）。

上面我们谈及了对 Row-Key 的设计，接着我们需要想想是否 Column Family 也会在不同的场景需要不同的设计方案呢。答案是肯定的，不过 CF 跟 Row-key 比较的话，确实也简单一些，但这并不意味着 CF 的设计就是一个琐碎的话题。在 RDBMS（传统关系数据库）系统中，我们知道如果当用户的信息分散在不同的表中，便需要根据一个 Key 进行 Join 操作。而在**HBase 中，我们需要设计 CF 来聚合用户所有相关信息**。简单来说，就是需要将数据按类别（或者一个特性）聚合在一个或多个 CF 中。这样，便可以根据 CF 获取这类信息。上面，我们讲解过一个 Region 对应于一个 CF。那么设想，如果在一个表中定义了多个 CF 时，就必然会有多个 Region。当 Client 查询数据时，就不得不查询多个 Region。这样性能自然会有所下降，尤其当 Region 跨机器查询的时候。因此在大多数的情况下，**一个表格不会超过 2 到 3 个 CF，而且很多情况下都是 1 个 CF 就足够了**。

## Phoenix 的使用

当一个新业务需要使用 HBase 时，是**完全可以使用 Java API 开发 HBase 的应用，从而实现具体的业务逻辑**。但是如果对于习惯使用 RDBMS 的 SQL，或者想要将原来使用 JDBC 的应用直接迁移到 HBase，这就是不可能的。由于这种缅怀过去的情怀，便催生了 Phoenix 的诞生。那么 Phoenix 都能提供哪些功能呢？**简单来说 Phoenix 在 HBase 之上提供了 OLTP 相关的功能，例如完全的 ACID 支持、SQL、二级索引等**，此外 Phoenix 还提供了标准的 JDBC 的 API。在 Phoenix 的帮助下，RDBMS 的用户可以很容易的使用 HBase，并且迁移原有的业务到 HBase 之中。下来就让我们简单了解一下，如何在 HBase 之上使用 Phoenix。

首先我们需要在 Phoenix 的网站下载与 HBase 版本对应的 Phoenix 安装包。我环境的 HBase 是通过 Ambari HDP2.4 部署的，其中的 HBase 是 1.1 的版本，因此我下载的是下图中的 phoenix-4.7.0-HBase-1.1。
