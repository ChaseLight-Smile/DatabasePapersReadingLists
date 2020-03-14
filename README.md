### 数据库论文阅读目录
 
#### 概述

该仓库将会介绍一些数据库、分布式系统领域的经典论文，包括SIGMOD、VLDB、ICDE、ATC、SOSP、ASPLOS、PPoPP、FAST、EuroSys、OSDI、NSDI、MOCRO、ISCA，按照自己的理解点评（总结介绍）一些论文的思路和创新点。

#### **论文介绍**

* index
    * SuRF: Practical Range Query Filtering with Fast Succinct Tries<br>
2018 sigmod best paper，该论文出自andy pavlo的学生。andy在数据库方面有独特的见解。bloom filter在disk-oriented database management system中有特别
的作用：在内存快读判断一个key是否存在，这些key就是存储的磁盘上的数据本身。该结构存在两个主要问题：（1）bloom filter存在“one-side error”也就是说：if key
is present, then bloom filter returns true. 该命题的逆否命题是：如果bloom filter返回false，那么说明key一定不存在。（2）bloom filter只支持point query，
但是不支持range query。针对bloom filter存在的第一个问题，要从设计合适的hash function等角度入手解决，比较难。针对第二个问题，SuRF采用了FST数据结构解决了
不支持范围查询的问题，能够给出开区间[key, +inf）,[key1, key2], (-inf, +inf)上的range query问题。


* Architecture
    * To BLOB or Not To BLOB: Large Object Storage in a Database or a Filesystem?<br>
本文主要讲述对于Binary Lager OBject（BLOB）究竟应该存储在database中使用DBMS管理还是应该用Filesystem来管理，即究竟BLOB应该作为一个数据库record还是file？
文章支持当文件系统是NTFS，DBMS是SQL Server 2005时，当文件大小下雨256KB时，DBMS管理更加高效，当对象大于1MB时，采用filesystem管理更加高效。文章还提出数据库
设计者应该考虑加入文件系统的碎片整理过程。**注：现今，当我们在存储视频等比较大的文件时，尽量不要直接用BLOB存储在数据库中，而是应该用filesystem管理，在数据库中可用只存储文件链接。**

    * Architecture of database system <br>
数据库体系结构最好的文章。作者图灵奖得主Stonebraker。包括processes model、query processing、storage and transaction、others，其中的事务存储部分严格区分了lock和latch，一个是higher level的概念，一个是lower lavel的概念。这两个改变在实现buffer pools时非常重要，latch在使用上，实际上用mutex来实现。lock是higher level事务层面用到的抽象。
    
    * The Snowflake Elastic Data Warehouse  (sigmod 2016)<br>
本文是一种标准的shared disk的架构。体现在两个方面：1、（1）整体架构分为三层：Storage layer由Amazon S3提供，仅仅能够提供put、get、delete操作，并且put操作只能是全量（in full）操作。get操作可以指定文件读取的范围。（2）stateless VM是，之所以说该层是stateless是因为：实际上该层的disk是cache的作用，用于系统的冷启动问题。该层可以和storage层独立扩展。（3）cloud service层，该层提供传统的数据库服务：事务管理、元数据管理（元数据持久到S3上），查询解析，查询优化等数据库传统的功能，但是不提供索引。列存数据库的列其实本身就是索引。底层为了避免全文件扫描，还实现了zone map（google、oracle对zone map可能拥有商标权）可以实现skip scan。该层也是stateless。2、online-upgrade时，实际上VMs层也是shard disk的架构，这样能够保证不同版本的系统在读取数据时，能够访问到相同的热数据。snonflaker实现了很多数据库中的经典技术，如MVCC并发控制方法（请注意MVCC不是并发控制协议，它需要和其它的如2PC、TO结合才能称为协议）、time travelling这是数据仓库中比较有用的技术，能够访问指定时间段的数据，snowflake指定只能访问90天以内的数，不过你能够自行实现backup、基于MVCC的SI隔离级别，在此基础之上保证了ACID事务等等。

    * How to copy files  (FAST 2020) <br>
    * MapReduce：Simplified Data Processing on Large Clusters<br>
    * GFS: The Google file system<br>
    * Bigtable: A Distributed Storage System for Structured Data<br>
    * Dynamo: Amazon’s Highly Available Key-value Store<br> 
    * Spanner: Google’s Globally-Distributed Database<br>
    * Spanner: Becoming a SQL System<br>
    * F1: A Distributed SQL Database That Scales<br>
    * Amazon Aurora: Design Considerations for High Throughput Cloud-Native Relational Databases<br>
    * (SPARK)Resilient Distributed Datasets：A Fault-Tolerant Abstraction for In-Memory Cluster Computing<br>
    * HadoopDB: An Architectural Hybrid of MapReduce and DBMS Technologies for Analytical Workloads or Integration of LargeScale Data Processing Systems and Traditional Parallel Database Technology<br>


* consistency or (consensus)
    * Paxos Made Simple<br>
    * raft: In Search of an Understandable Consensus Algorithm <br>
    * (raft-ATC)In Search of an Understandable Consensus Algorithm
    * Strong and Efficient Consistency with Consistency-Aware Durability (FAST 2020 best paper)<br>
该篇文章是FAST 2020 best paper，主要讲述了在分布式存储系统中durable数据时，如何在保证strong consistency的前提下，实现higher throughput 和 lower latency，在传统的强一致性模型下，由于每一次数据修改操作都需要immediately将数据从master同步到follower上，这个过程在分布式场景下，会严重降低系统性能。本文作者观察到：在强一直模型下，如果能够保证一个cross-client 单调读的特性，那么我们就可以采用乐观的方式进行写后的读操作。本文的的方法我认为叫做optimistic read会更加的贴切，当我某个出follower中的数据时，会和durable index比较，就能够得出是否当前该数据版本是最新的，如果不是，那么系统就会在此时锁定将数据从主节点复制过来并且完成持久化，并返回结果给客户端；如果是最新的，那么字节返回给客户端结果。乍看来，这样子系统的性能岂不是会很差？但是其实不是，因为就像乐观并发控制协议一样，作者假设大部分情况下分布式系统的后台进程会将数据从master持久到replicas中，并且在存在follower的情况下，你请求到replica，并且这个数据恰好不是最新的可能性不大，**除非系统在频繁更新数据，并且也在频繁读取更新过的数据，这样性能的性能会发生抖动**。本人还解决了存在网络分区情况下的数据的读取，如果存在网络分区，那么系统会原子地保证分区节点从active set中剔除出去，这样在分区节点上的读操作就会被锁定，不允许从这个节点读取数据，等待分区情况解决之后，会重新持久最新的数据到分区节点上，并将其加入到active set中，保证单调读特性。本文的实现很优美，我觉得是durable和consistency实现中的一股清流。


* query processing or query plan
    * Encapsulation of Parallehsm in the Volcano Query Procesing System <br>
    * LEO: DB2’s LEarning Optimizer <br>
    
* recovery
    * aries: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging<br>
ARIES是1992年的文章，目前所有的数据库管理系统都采用ARIES算法作故障恢复，但是每个系统的实现可能略微有些差距。ARIES依赖WAL日志，并且规定数据真正写入到磁盘之前，日志必须首先写入磁盘，其引入了pageLSN，recoLSN、flushLSN、MasterRecord概念，使得log能够完整的追踪整个日志的状态，还提出了CLR等强有力的日志回滚机制，能够保证在log中如何实现回滚操作。LSN（log sequence number）是一个强大的机制，它本身是一个单调递增的counter，如何实现这个counter是一个比较重要的技术，尤其在分布式数据库中。很多当前最新的与事务相关的文章都借鉴了这个思想，比如Strong and Efficient Consistency with Consistency-Aware Durability (FAST 2020 best paper)中的durable index和persist index、update index也都是借鉴了log的LSN机制。该文较长，并且数学抽象较好，需要花费不少时间才能完全理解。
    

* serverless
    * Catalyzer: Sub-millisecond Startup for Serverless Computing withInitialization-less Booting.(ASPLOS 2020)<br>
文章主要为了降低serverless场景下应用启动的时间，作者认为该时间应该分为三个部分：kernel启动时间；application依赖环境启动时间；application运行时间，除了app运行时间以外的两个时间都应该降下来，通过设计了合理的image机制，采用mmap将image映射到内存中，并建立合适的template，加速整个启动过程。第一篇serverless的文章，比较粗浅，希望以后多看一些这方面内容吧。<br>
    * Cloud Programming Simplified: A Berkeley View on Serverless Computing <br>
该文是serverless最基础论文。

#### 参考资料
* https://github.com/JunpengCode/databaseology
* https://github.com/JunpengCode/awesome-database-learning
* https://www.kawabangga.com/db
* http://www.cs.cmu.edu/afs/cs/academic/class/15721-f01/www/readings.html
* http://pages.cs.wisc.edu/~ra/Classes/739-sp20/readings.html
* FAST/NSDI https://www.usenix.org/publications/proceedings?page=2
