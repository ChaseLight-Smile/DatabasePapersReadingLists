### 数据库论文阅读目录
 
#### 概述

该仓库将会介绍一些数据库领域的经典论文，包括SIGMOD、VLDB、ICDE、FAST、ATC、NSDI中的best paper，分组了部分论文，根据自己的理解，可能不全面。

#### **论文介绍**

* index
    * SuRF: Practical Range Query Filtering with Fast Succinct Tries
2018 sigmod best paper，该论文出自andy pavlo的学生。andy在数据库方面有独特的见解。bloom filter在disk-oriented database management system中有特别
的作用：在内存快读判断一个key是否存在，这些key就是存储的磁盘上的数据本身。该结构存在两个主要问题：（1）bloom filter存在“one-side error”也就是说：if key
is present, then bloom filter returns true. 该命题的逆否命题是：如果bloom filter返回false，那么说明key一定不存在。（2）bloom filter只支持point query，
但是不支持range query。针对bloom filter存在的第一个问题，要从设计合适的hash function等角度入手解决，比较难。针对第二个问题，SuRF采用了FST数据结构解决了
不支持范围查询的问题，能够给出开区间[key, +inf）,[key1, key2], (-inf, +inf)上的range query问题。


* Architecture
    * To BLOB or Not To BLOB: Large Object Storage in a Database or a Filesystem?
本文主要讲述对于Binary Lager OBject（BLOB）究竟应该存储在database中使用DBMS管理还是应该用Filesystem来管理，即究竟BLOB应该作为一个数据库record还是file？
文章支持当文件系统是NTFS，DBMS是SQL Server 2005时，当文件大小下雨256KB时，DBMS管理更加高效，当对象大于1MB时，采用filesystem管理更加高效。文章还提出数据库
设计者应该考虑加入文件系统的碎片整理过程。**注：现今，当我们在存储视频等比较大的文件时，尽量不要直接用BLOB存储在数据库中，而是应该用filesystem管理，在数据库中可用只存储文件链接。**

    * Architecture of database system 数据库体系结构最好的文章。作者图灵奖得主Stonebraker。包括processes model、query processing、storage and transaction、others，其中的事务存储部分严格区分了lock和latch，这两个改变在实现buffer pools时非常重要，latch在使用上，实际上用mutex来实现。

* consistency or (consensus)
    * Paxos Made Simple
    * Strong and Efficient Consistency with Consistency-Aware Durability (FAST 2020 best paper)
该篇文章是FAST 2020 best paper，主要讲述了在分布式存储系统中durable数据时，如何在保证strong consistency的前提下，实现higher throughput 和 lower latency，在传统的强一致性模型下，由于每一次数据修改操作都需要immediately将数据从master同步到follower上，这个过程在分布式场景下，会严重降低系统性能。本文作者观察到：在强一直模型下，如果能够保证一个cross-client 单调读的特性，那么我们就可以采用乐观的方式进行写后的读操作。本文的的方法我认为叫做optimistic read会更加的贴切，当我某个出follower中的数据时，会和durable index比较，就能够得出是否当前该数据版本是最新的，如果不是，那么系统就会在此时锁定将数据从主节点复制过来并且完成持久化，并返回结果给客户端；如果是最新的，那么字节返回给客户端结果。乍看来，这样子系统的性能岂不是会很差？但是其实不是，因为就像乐观并发控制协议一样，作者假设大部分情况下分布式系统的后台进程会将数据从master持久到replicas中，并且在存在follower的情况下，你请求到replica，并且这个数据恰好不是最新的可能性不大，**除非系统在频繁更新数据，并且也在频繁读取更新过的数据，这样性能的性能会发生抖动**。本人还解决了存在网络分区情况下的数据的读取，如果存在网络分区，那么系统会原子地保证分区节点从active set中剔除出去，这样在分区节点上的读操作就会被锁定，不允许从这个节点读取数据，等待分区情况解决之后，会重新持久最新的数据到分区节点上，并将其加入到active set中，保证单调读特性。本文的实现很优美，我觉得是durable和consistency实现中的一股清流。

#### 参考资料
* https://github.com/JunpengCode/databaseology
* https://github.com/JunpengCode/awesome-database-learning
* https://www.kawabangga.com/db
* http://www.cs.cmu.edu/afs/cs/academic/class/15721-f01/www/readings.html
* http://pages.cs.wisc.edu/~ra/Classes/739-sp20/readings.html
* FAST/NSDI https://www.usenix.org/publications/proceedings?page=2
