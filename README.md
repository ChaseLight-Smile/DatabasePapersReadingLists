### 数据库论文阅读目录

![Github stars](https://img.shields.io/github/stars/JunpengCode/DatabasePapersReadingLists.svg) ![](https://img.shields.io/github/license/JunpengCode/DatabasePapersReadingLists.svg) ![Github stars](https://img.shields.io/github/forks/JunpengCode/DatabasePapersReadingLists.svg)

 
#### 概述

该仓库将会介绍一些数据库、分布式系统领域的经典论文，包括SIGMOD、VLDB、ICDE、ATC、SOSP、ASPLOS、PPoPP、FAST、EuroSys、OSDI、NSDI、MOCRO、ISCA，
按照自己的理解总结一些论文的思路和创新点。**每日一篇，never stop！**

#### **论文介绍**


* Index
    * SuRF: Practical Range Query Filtering with Fast Succinct Tries<br>
2018 sigmod best paper，该论文出自andy pavlo的学生。andy在数据库方面有独特的见解。bloom filter在disk-oriented database management system中有特别的作用：在内存快读判断一个key是否存在，这些key就是存储的磁盘上的数据本身。该结构存在两个主要问题：（1）bloom filter存在“one-side error”也就是说：if key is present, then bloom filter returns true. 该命题的逆否命题是：如果bloom filter返回false，那么说明key一定不存在。（2）bloom filter只支持point query，但是不支持range query。针对bloom filter存在的第一个问题，要从设计合适的hash function等角度入手解决，比较难。针对第二个问题，SuRF采用了FST数据结构解决了不支持范围查询的问题，能够给出开区间[key, +inf）,[key1, key2], (-inf, +inf)上的range query问题。
	* SAL-Hashing： A Self-Adaptive Linear Hashing Index for SSDs<br>
	* The Case for Learned Index Structures<br>
Google在系统索引与Deep Learning结合的研究，ML for Sytstem是当前的研究热点。缺点是不能执行update操作。
		* ALEX: An Updatable Adaptive Learned Index<br>
上篇文章的作者改进Update操作的文章。
		* Learning Multi-dimensional Indexes<br>
上篇文章作者在多维索引上的工作进展。
		* XIndex: A Scalable Learned Index for Multicore Data Storage<br>
			* XIndex： A Scalable Learned Index for Multicore Data Storage - extention <br>
上海交通大学陈海波组在学习型索引上的研究进展。
    * The Ubiquitous B-Tree<br>
    * BzTree: A High-Performance Latch-free Range Index for Non-Volatile Memory<br>
    * Modern B-Tree Techniques<br>
    * The Bw-Tree: A B-tree for New Hardware Platforms<br>
    * Efficient Locking for Concurrent Operations On B-Trees<br>
CMU大佬巨作 B-Link tree。
    * Easy Lock-Free Indexing in Non-Volatile Memory<br>
    * Persistent Bloom Filter Membership Testing for the Entire History
	* LSM-based Storage Techniques: A Survey</br>
该文的特点是简单易懂。
		* Evaluation of High Performance Key-Value Stores</br>
学习leveldb很好的指导论文。

* Sorting
	* Implementing Sorting in Database Systems</br>
Goetz Graefe老爷子大作，他写了关于B-tree、Volcano查询处理模型等。

* Architecture
    * To BLOB or Not To BLOB: Large Object Storage in a Database or a Filesystem?<br>
本文主要讲述对于Binary Lager OBject（BLOB）究竟应该存储在database中使用DBMS管理还是应该用Filesystem来管理，即究竟BLOB应该作为一个数据库record还是file？
文章支持当文件系统是NTFS，DBMS是SQL Server 2005时，当文件大小下雨256KB时，DBMS管理更加高效，当对象大于1MB时，采用filesystem管理更加高效。文章还提出数据库
设计者应该考虑加入文件系统的碎片整理过程。**注：现今，当我们在存储视频等比较大的文件时，尽量不要直接用BLOB存储在数据库中，而是应该用filesystem管理，在数据库中可用只存储文件链接。**

    * Architecture of database system <br>
数据库体系结构最好的文章。作者图灵奖得主Stonebraker。包括processes model、query processing、storage and transaction、others，其中的事务存储部分严格区分了lock和latch，一个是higher level的概念，一个是lower lavel的概念。这两个改变在实现buffer pools时非常重要，latch在使用上，实际上用mutex来实现。lock是higher level事务层面用到的抽象。
    
    * The Snowflake Elastic Data Warehouse  (sigmod 2016)<br>
本文是一种标准的shared disk的架构。体现在两个方面：1、（1）整体架构分为三层：Storage layer由Amazon S3提供，仅仅能够提供put、get、delete操作，并且put操作只能是全量（in full）操作。get操作可以指定文件读取的范围。（2）stateless VM是，之所以说该层是stateless是因为：实际上该层的disk是cache的作用，用于系统的冷启动问题。该层可以和storage层独立扩展。（3）cloud service层，该层提供传统的数据库服务：事务管理、元数据管理（元数据持久到S3上），查询解析，查询优化等数据库传统的功能，但是不提供索引。列存数据库的列其实本身就是索引。底层为了避免全文件扫描，还实现了zone map（google、oracle对zone map可能拥有商标权）可以实现skip scan。该层也是stateless。2、online-upgrade时，实际上VMs层也是shard disk的架构，这样能够保证不同版本的系统在读取数据时，能够访问到相同的热数据。snonflaker实现了很多数据库中的经典技术，如MVCC并发控制方法（请注意MVCC不是并发控制协议，它需要和其它的如2PC、TO结合才能称为协议）、time travelling这是数据仓库中比较有用的技术，能够访问指定时间段的数据，snowflake指定只能访问90天以内的数，不过你能够自行实现backup、基于MVCC的SI隔离级别，在此基础之上保证了ACID事务等等。

    * FlashKV Accelerating KV Performance with Open-Channel SSDs (2017)<br>
    * MapReduce：Simplified Data Processing on Large Clusters<br>
MapReduce一文确实是一篇极好的文章，按照其思想，它的输入输出全部存储在GFS（开源实现下，将数据存储在HDFS之上）上。从用户的角度看，不需要考虑load balance、communication等，只需要写业务
端程序，也就是只需要设计合适的map和reduce函数即可，当然为了减少network communication代价，也可以写combiner function，实际上map和reduce阶段的操作就是两次hash。map tasks+reduce tasks= job，
也就是说多个map tasks 和 多个reduce tasks构成一个job，map生成的intermmidiate key/values会存储在当前map节点的non-volatile存储设备上。接着master会调度将这些结果重新shuffle，然后将结果
调度给reduce tasks，尽量保证不同reduce任务之间减少network communication。
    * GFS: The Google file system<br>
Google经典三篇文章之一，GFS是google真正对工业界在数以千计的廉价商业化机器上实现的分布式文件系统，其设计包含一个master节点（为了保证reliable等特性，该机器通常会有replicas），一次写操作
成功不仅需要在所有的chunkserver上写入数据，还需要master节点的operation logs落盘，master节点与传统单机DBMS的设计一样。master节点不真实存储任何client数据，client数据全部存储在chunkserver上，
master节点保存三件事情：（1）filename -> array of chunk handler, （2）chunkhandler -> list of chunkserver, （3）logs and checkpoints，可以看出第一点和第二点主要是为了能够找到文件的真实位置，
第三点主要是为了实现可靠性和可恢复性。当client节点将请求发送到master之后，master会将用户请求的所在的chunkserver位置发送给client，并且client端会缓存这些信息，以便于当client再次请求相同的数据时，
不需要再次请求master，那么此时就有一个数据的一致性问题，一旦映射关系发生变化（比如某个chunkserver fails），这个cache信息就会失效，或者设计一个lease，给定时间窗口内有效，一旦过时cache就立即
停止服务。在GFS中假设发生追加写操作的可能性远远大于随机写操作。
    * Bigtable: A Distributed Storage System for Structured Data<br>
Google经典三篇文章之一，在Bigtable基础之上，Google实现了leveldb。bigtable实际上是一张”宽表“，我们知道，在经典的关系数据库范式理论中，首先数据库的表设计应该满足第一范式，也就是表中的属性不能再分，
但是Bigtable使用了column family的特性，使得表的列属性实际上是可分的，同一个column family中可以有多个不同的列，这样子构成了bigtable的宽表模式。row-key按照字典序排序，并且row-key组合成了bigtable的tablet，每个tablet中
可能包含很多个row-key，然后这些在内存中被组织成为memtale，当memtable达到一定的大小以后，就变为immutable memtable，接着被compaction成为SSTable，SSTtable也会被major compaction，当生成更大的SSTable之后，就会删除
原来的SSTable，所以这个过程实际上存在非常多的写放大。为了出现fails时能够恢复，还存在了redo log，这一设计完全是从关系数据库的ARIES中迁移而来。最后还介绍了不少优化，我认为想要真正读懂这个文章
的所有优化，尽量结合着leveldb的源码，并做一些实验。
    * Dynamo: Amazon’s Highly Available Key-value Store<br> 
Dynamo是Amazon在2007年SOSP上发表的关于键值对存储的分布式系统，主要强调高可用、强扩展，使用读操作之后的最终一致性方案，这在2020年看来并不是一个好的选择，
强一致性仍然是当前的需求。但是这不影响Dynamo成为高引用文章，其中很多的技术都成为很多数据库设计的规范。Dynamo的强扩展性（scale-out）采用了consistent hashing，
当在其中增加或者删除node时，不要执行reshuffle（rehash）的操作，当然consistent hasing不是Dynamo提出的技术，早在2000年就提出额consistent hashing。
读操作和写操作都松散的希望能得到系统中至少半数点的回应，但是考虑到可能存在网络分区等错误，所以仅仅是一个松散的建议，在写操作时，如果某个ring中的节点失去了联系，
那么就讲备份数据写到下一个ring中，并记录好，等待该节点恢复后，再将数据拷贝过去。
Dynamo在读操作也希望能有半数以上的节点返回数据，读出的数据会带有数据版本（文章中称为vector clock），如果发现数据不一致，允许在系统层面进行规约，但是不强制，
因为Dynamo允许业务逻辑层处理数据的不一致性（比如在Amazon中，用户的购物车可以由用户自己来维护其一致性）。其中Grossip-based的协议实现信息在节点中的传播，
这些古老的技术都在Dynamo得到了很好的应用。可以说Dynamo是结合了很多优秀实现技术的一个原型产品，堪称教科书式的实现。
	* (SPARK)Resilient Distributed Datasets：A Fault-Tolerant Abstraction for In-Memory Cluster Computing<br>
    * Granularity of Locks and Degrees of Consistency in a Shared Data Base<br>
jim gray首次在提出granularity of lockable object，也就是被锁对象的粒度，粒度这个词很抽象，在这里granularity = size指的是被锁对象的大小。该文提出了intension lock，使得在系统的并发度和overhead of lock manager
上做了trade-off，本质上，在被锁对象层次结构下，意向锁的提出了解决了这样两个问题：（1）如果low level层次对象加了锁，那么其祖先节点现在想要做某个查询，如何快速判断是否能够获取到锁执行相应的操作？（2）如果high level
节点获取了某种锁，接着另外一个事务想要在其孩子节点上做某些操作，是否能够直接获取到某种锁并执行相应的操作？因为本文提出了意向锁+root到leaf的加锁顺序，规范了整个lock protocol。
	* Spark SQL: Relational Data Processing in Spark<br>
	* Spanner: Google’s Globally-Distributed Database<br>
    * Spanner: Becoming a SQL System<br>
    * F1: A Distributed SQL Database That Scales<br>
    * Amazon Aurora: Design Considerations for High Throughput Cloud-Native Relational Databases<br>
    * HadoopDB: An Architectural Hybrid of MapReduce and DBMS Technologies for Analytical Workloads or Integration of LargeScale Data Processing Systems and Traditional Parallel Database Technology<br>
    * Soft Updates: A Solution to the Metadata Update Problem in File Systems<br>
    * Rethinking Database High Availability with RDMA Networks<br>
    * CockroachDB: The Resilient Geo-Distributed SQL Database (SIGMOD 2020)
	* C-Store: A Column-oriented DBMS<br>
* Consistency or (consensus)
    * Paxos Made Simple<br>
    * raft: In Search of an Understandable Consensus Algorithm <br>
    * (raft-ATC)In Search of an Understandable Consensus Algorithm<br>
    * Linearizability : A Correctness Condition for Concurrent Object<br>
    * Sequential consistency versus linearizability<br>
    * Strong and Efficient Consistency with Consistency-Aware Durability (FAST 2020 best paper)<br>
该篇文章是FAST 2020 best paper，主要讲述了在分布式存储系统中durable数据时，如何在保证strong consistency的前提下，实现higher throughput 和 lower latency，在传统的强一致性模型下，由于每一次数据修改操作都需要immediately将数据从master同步到follower上，这个过程在分布式场景下，会严重降低系统性能。本文作者观察到：在强一直模型下，如果能够保证一个cross-client 单调读的特性，那么我们就可以采用乐观的方式进行写后的读操作。本文的的方法我认为叫做optimistic read会更加的贴切，当我某个出follower中的数据时，会和durable index比较，就能够得出是否当前该数据版本是最新的，如果不是，那么系统就会在此时锁定将数据从主节点复制过来并且完成持久化，并返回结果给客户端；如果是最新的，那么直接返回给客户端结果。乍看来，这样子系统的性能岂不是会很差？但是其实不是，因为就像乐观并发控制协议一样，作者假设大部分情况下分布式系统的后台进程会将数据从master持久到replicas中，并且在存在follower的情况下，你请求到replica，并且这个数据恰好不是最新的可能性不大，**除非系统在频繁更新数据，并且也在频繁读取更新过的数据，这样性能的性能会发生抖动**。本人还解决了存在网络分区情况下的数据的读取，如果存在网络分区，那么系统会原子地保证分区节点从active set中剔除出去，这样在分区节点上的读操作就会被锁定，不允许从这个节点读取数据，等待分区情况解决之后，会重新持久最新的数据到分区节点上，并将其加入到active set中，保证单调读特性。本文的实现很优美，我觉得是durable和consistency实现中的一股清流。
	* Hermes: a Fast, Fault-Tolerant and Linearizable Replication Protocol<br>
该篇文章出自ASPLOS'2020，基于广播的member-ship协议，时间复杂度为O(logn)，和Dynamo中的gossip-based确认ring中节点是否alive方法很像，实现比较经典。
* Time, Clocks and Ordering of events
	* Time, Clocks, and the Ordering of Events in a Distributed System</br>
论文主要讲述了Lamport Clocks，并且区别了time-of-day（wall） clock， monotonic clock, and logical clock inicluding Lamport clock and vector clock。
    * detecting causal relationships in distributed computations in search of the holy grail</br>
vector clock在分布式系统中是一个更重要的概念，可以用来解释happened before relations、causal order and FIFO order。
* Lock-free data structure
    * Lock-Free Data Structures<br>
对于熟悉多线程编程的programmer来说，大家很熟悉应该使用各种lock（shared lock、exclusive clock、unique lock）等锁定critical section部分，这种方法通常称为latch-full，使得可能发生race condition的critical section部分能够被串行化访问，实际上这是一种非常低效的访问方式，比如：当两个线程访问相同的B+-tree的叶子节点做范围扫描，并且两者分别向相对（如 ->  B+-tree leaves  <-）的方向访问，这样的情况下很容易造成死锁，我们需要更多的机制来避免这样的情况的发生，最简单的方法是在访问开始时，获得所有需要访问的资源的锁，当然这种实现是低效的，其中有包括2PC、S2PC等方法。但是实际上在多线程编程社区还有一种lock-free的编程模式，实际上我更倾向于叫latch-free，锁是一个较为higher level的结构，在ctitical section部分叫latch-free更加合适。本文主要是讲解如何应用latch-free技术实现基本的数据结构，近五年来（2015年至今）很多的DBMS实现技术中，大部分都开始采用latch-free的数据结构，比如andy pavlo的peleton等。在latch-free的编程世界中，你几乎不能原子的做大部分的操作，只有仅仅一小部分能够被原子的完成，这加大了latch-free编程的难度。但实际上从2004年开始世界上就陆续有很多人在接触latch-free技术。CoW、MVCC都是经典的lock-free的实现，并且由于实现这些技术的系统中都有很强的GC机制，所以使得lock-free的内存资源回收变得简单。在这篇文章中，最终实现了一种lock-free的map，但是实际上，在update操作中还是加了隐形的锁，在大量并发读操作请求下，导致update操作产生饥饿，本文没有能解决这个问题，在Lock-Free Data Structures with Hazard Pointers文章中，真正解决了lock-free结构实现下产生的update饥饿问题，并且整个实现非常优雅。本文给出了两种不是很优雅的lock-free的是方法：（1）采用DCAS（double Compare and Swap）原语，实际上在大部分的CPU指令中并没有实现，DCAS的本质是能够原子的交换两个指针的值；（2）基于两次CAS原语的map实现，该种实现方法能够解决lock-free存在的问题，但是带来了新的问题：写饥饿问题，也就是仅仅适合read not many的场景。
    * Lock-Free Data Structures with Hazard Pointers<br>
hazard pointer能够优雅的实现lock-free的内存回收，hazard pointer实际上是一个single-writer multi-reader shared pointer。这个指针会被reader thread拥有（reader writer会在要读的数据上分配一个hazard pointer），一旦reader pointer拥有了这个指针，就会告知其它writer thread我正在读这个数据，你可以替换它，但是你不能改变它的内容，也不能删除它（也就是说你只能读取到这个数据，但是writer thread不能做任何的修改操作，这对于writer thread来说也是一种progress。也可以说是保证了所有线程尽量wait-free）。hazard pointer是一个复杂的结构，它会保存系统中所有线程（all threads）的读请求中访问的存储区域，当一个写线程copy了一个新的存储区域的副本，并且做了修改之后成为了“新”的副本（也就是之后的读写操作应该建立在该副本之上），然后这个写线程必须要将旧的副本删除，这显然会导致一个极其危险的操作，因此，实际上hazard pointer要求在删除时，扫描整个hazard结构，做一个集合的差运算，如果这个要淘汰的旧的副本不在这个结构中，说明该副本能够被安全的删除，这就是其核心思想。至于hazard point可以采用链表来维护其结构，也可以采用hash table来维护，不同的结构，性能不同。
    * Hazard Pointers Safe Memory Reclamation for Lock-Free Objects<br>
hazard pointer(危险指针)，为什么叫做“危险指针”？本文给出了一些简单的理由：读线程将要读取的存储地址加入到所有线程的“危险指针”结构中，也就告诉其他线程，该读线程之后的所有读写内容的操作将不会对该存储地址进行任何验证。这个操作被认为是“危险”的。其他的线程在看到该存储内容是“危险”的，将不会删除或者重用该地址的内容。从代码中我们能够看出**将存储引用部分将入到hazard pointer中的动作必须要早于其他线程执行retired操作之前**。本文给出了hazard pointer如何解决ABA问题的证明，实际上在hazard pointer method中直接避免了ABA问题的发生。早期的文献已经证明，在存在GC的系统中一直是ABA-safe，本文证明在hazard pointer方法中也是ABA-safe。
    * Hazard Pointers: Safe Resource Reclamation for Optimistic Concurrency<br>
    * Wait-free synchronization<br>
    * C++ and the Perils of Double-Checked Locking <br>
    * How to Copy Files (FAST 2020)<br>
该文是如何优雅的实现lock-free的copy操作和内存区域deallocation操作当前最新的研究。采用了树形结构来维护一个hazard pointer结构，较早之前的研究都是采用了single-linked list 或者 hash table。本文第一作者非常擅长lock-free数据结构的设计与实现。
	* Latch-free Synchronization in Database Systems Silver Bullet or Fool's Gold<br>
图灵奖得主stonebreker的学生Abadi的文章，文章的一作是Abadi的博士，获得了2020 jim gray优秀博士论文奖，该文是对Latch-free的批判文章。我认为本文存在的问题是：立意不正确。latch-based存在就是假设对临界资源的访问存在大量竞争，而latch-free存在本身就是假设对临界资源的访问竞争发生可能性很小。本文得出的结果是：high contention condition，latch-free不合适，这个结论显得过于苍白，甚至感觉有些滑稽。

* Query processing or query plan
    * Encapsulation of Parallehsm in the Volcano Query Procesing System <br>
    * LEO: DB2’s LEarning Optimizer <br>
	* Neo: A Learned Query Optimizer<br>
    * Review of Algorithms for the Join Ordering Problem in Database Query Optimization <br>
    
* Recovery
    * aries: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging<br>
ARIES是1992年的文章，目前所有的数据库管理系统都采用ARIES算法作故障恢复，但是每个系统的实现可能略微有些差距。ARIES依赖WAL日志，并且规定数据真正写入到磁盘之前，日志必须首先写入磁盘，其引入了pageLSN，recoLSN、flushLSN、MasterRecord概念，使得log能够完整的追踪整个日志的状态，还提出了CLR等强有力的日志回滚机制，能够保证在log中如何实现回滚操作。LSN（log sequence number）是一个强大的机制，它本身是一个单调递增的counter，如何实现这个counter是一个比较重要的技术，尤其在分布式数据库中。很多当前最新的与事务相关的文章都借鉴了这个思想，比如Strong and Efficient Consistency with Consistency-Aware Durability (FAST 2020 best paper)中的durable index和persist index、update index也都是借鉴了log的LSN机制。该文较长，并且数学抽象较好，需要花费不少时间才能完全理解。
    * Why Do Computers Stop and What Can Be Done About It?</br>
https://www.hpl.hp.com/techreports/tandem/TR-85.7.pdf
	* Concurrency Control and Recovery [M. J. Franklin 1997]</br>
https://courses.cs.washington.edu/courses/cse544/11wi/papers/franklin97.pdf 本文对并发控制和恢复给出了一个详细的综述，并且涉及到了ACID的发展历史。对于并发控制和恢复存在很多的解决办法，
但是ACID模型将自己与其他模型分别开来，主要是由于两点：（1）在ACID模型中加入了隔离性和容错机制；（2）ACID模型将多个对不同对象的读写操作作为一个原子的、隔离的执行单元。ACID的这些方面都对数据
提供了强有力的保护。当然实现这些特性的代价是增加了系统实现的复杂性，并且也产生了性能代价（其实就是一定程度上降低了并发度，但是保证了系统执行的逻辑正确性）。这些特性最终给使用DBMS的用户提供
高可用和可靠性。本文也给出了“notion of correctness for concurrenct execution of transactions is serializability”，其中conflict serializability是最广泛被接受的事务正确性的概念。因为冲突可串行化
不论是检测（detect）还是实现（enforce）都很高效并且容易实现。另外一个被广泛接受是view serializability，它比conflict serializability有更少的限制，因此会得到更多合法的调度结果。但是视图可串行化需要一个
“上帝视角”，因此其只有理论价值，这主要是因为他们实现起来非常困难。总之，本文是非常好的综述，具体阅读笔记参见我的ppt.
	*  Critique of ANSI SQL Isolation Levels<br>
https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf 详细描述了基于不同concurrency control protocol下可能出现的anomoly以及对应的解决办法。我们平常所见的dirty read/unrepeatable read/phantom都是在
基于locking的并发控制方法下得到的，在乐观并发方法和基于mvcc的并发方法中存在其他的异常，请参考本文来规范对隔离级别和一致性的trade-off。

* Serverless
    * Catalyzer: Sub-millisecond Startup for Serverless Computing withInitialization-less Booting.(ASPLOS 2020)<br>
文章主要为了降低serverless场景下应用启动的时间，作者认为该时间应该分为三个部分：kernel启动时间；application依赖环境启动时间；application运行时间，除了app运行时间以外的两个时间都应该降下来，通过设计了合理的image机制，采用mmap将image映射到内存中，并建立合适的template，加速整个启动过程。第一篇serverless的文章，比较粗浅，希望以后多看一些这方面内容吧。<br>
    * Cloud Programming Simplified: A Berkeley View on Serverless Computing <br>
该文是serverless最基础论文。

* In-memory database
	* Survey
		* Main memory database systems：an overview (H.G. Molina文章)<br>
文章发表在1992年，但是对于main-memory问题的总结在今年看来仍然很适用，对比今天的main-memory db也都基本讨论了本文提出的所有问题，非常值得精读。对in-memory中transaction、data storage、
index、cache locality、log、lock manager都有比较经典的总结和思考。
	* Transactions
		* Staring into the Abyss: An Evaluation of Concurrency Control with One Thousand Cores<br>
该文发表在2014年VLDB上，文章中主要揭示in-memory database在面临hundred of cores时，什么样的并发控制协议性能最好，扩展性最强，文章的结论是：目前存在的并发控制协议，都有局限性。
因此对于当今硬件，应该from ground up进行software and hardware codesign。
	* Architecture
		* NVRAMaware Logging in Transaction Systems</br>
		* Let's Talk About Storage & Recovery Methods for Non-Volatile Memory Database Systems</br>

#### 参考资料

* https://www.kawabangga.com/db
* http://www.cs.cmu.edu/afs/cs/academic/class/15721-f01/www/readings.html
* http://pages.cs.wisc.edu/~ra/Classes/739-sp20/readings.html
* FAST/NSDI https://www.usenix.org/publications/proceedings?page=2
* Sigmod 2020 papers https://dl.acm.org/doi/proceedings/10.1145/3403425
* crowdsourcing https://research.yandex.com/tutorials/crowd/kdd-2019
* http://coding-geek.com/how-databases-work/
