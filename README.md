### 数据库论文阅读目录

![Github stars](https://img.shields.io/github/stars/JunpengCode/DatabasePapersReadingLists.svg) ![](https://img.shields.io/github/license/JunpengCode/DatabasePapersReadingLists.svg) ![Github stars](https://img.shields.io/github/forks/JunpengCode/DatabasePapersReadingLists.svg) ![Github Release](https://img.shields.io/github/release/JunpengCode/DatabasePapersReadingLists.svg)

 
#### 概述

该仓库将会介绍一些数据库、分布式系统领域的经典论文，包括SIGMOD、VLDB、ICDE、ATC、SOSP、ASPLOS、PPoPP、FAST、EuroSys、OSDI、NSDI、MOCRO、ISCA，按照自己的理解总结一些论文的思路和创新点。

#### **论文介绍**


* index
    * SuRF: Practical Range Query Filtering with Fast Succinct Tries<br>
2018 sigmod best paper，该论文出自andy pavlo的学生。andy在数据库方面有独特的见解。bloom filter在disk-oriented database management system中有特别的作用：在内存快读判断一个key是否存在，这些key就是存储的磁盘上的数据本身。该结构存在两个主要问题：（1）bloom filter存在“one-side error”也就是说：if key is present, then bloom filter returns true. 该命题的逆否命题是：如果bloom filter返回false，那么说明key一定不存在。（2）bloom filter只支持point query，但是不支持range query。针对bloom filter存在的第一个问题，要从设计合适的hash function等角度入手解决，比较难。针对第二个问题，SuRF采用了FST数据结构解决了不支持范围查询的问题，能够给出开区间[key, +inf）,[key1, key2], (-inf, +inf)上的range query问题。
	* SAL-Hashing： A Self-Adaptive Linear Hashing Index for SSDs<br>
	* The Case for Learned Index Structures<br>
Google在系统索引与Deep Learning结合的研究，ML for Sytstem是当前的研究热点。缺点是不能执行update操作。
		* ALEX: An Updatable Adaptive Learned Index<br>
上篇文章的作者改进Update操作的文章。
		* Learning Multi-dimensional Indexes<br>
上篇文章作者在多维索引上的工作进展。

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
    * GFS: The Google file system<br>
    * Bigtable: A Distributed Storage System for Structured Data<br>
    * Dynamo: Amazon’s Highly Available Key-value Store<br> 
Dynamo是Amazon在2007年SOSP上发表的关于键值对存储的分布式系统，主要强调高可用、强扩展，使用读操作之后的最终一致性方案，这在2020年看来并不是一个好的选择，强一致性仍然是当前的需求。但是这不影响Dynamo成为高引用文章，其中很多的技术都成为很多数据库设计的规范。Dynamo的强扩展性（scale-out）采用了consistent hashing，当在其中增加或者删除node时，不要执行reshuffle（rehash）的操作，当然consistent hasing不是Dynamo提出的技术，早在2000年就提出额consistent hashing。读操作和写操作都松散的希望能得到系统中至少半数点的回应，但是考虑到可能存在网络分区等错误，所以仅仅是一个松散的建议，在写操作时，如果某个ring中的节点失去了联系，那么就讲备份数据写到下一个ring中，并记录好，等待该节点恢复后，再将数据拷贝过去。Dynamo在读操作也希望能有半数以上的节点返回数据，读出的数据会带有数据版本（文章中称为vector clock），如果发现数据不一致，允许在系统层面进行规约，但是不强制，因为Dynamo允许业务逻辑层处理数据的不一致性（比如在Amazon中，用户的购物车可以由用户自己来维护其一致性）。其中Grossip-based的协议实现信息在节点中的传播，这些古老的技术都在Dynamo得到了很好的应用。可以说Dynamo是结合了很多优秀实现技术的一个原型产品，堪称教科书式的实现。


    * Spanner: Google’s Globally-Distributed Database<br>
    * Spanner: Becoming a SQL System<br>
    * F1: A Distributed SQL Database That Scales<br>
    * Amazon Aurora: Design Considerations for High Throughput Cloud-Native Relational Databases<br>
    * (SPARK)Resilient Distributed Datasets：A Fault-Tolerant Abstraction for In-Memory Cluster Computing<br>
    * HadoopDB: An Architectural Hybrid of MapReduce and DBMS Technologies for Analytical Workloads or Integration of LargeScale Data Processing Systems and Traditional Parallel Database Technology<br>
    * Soft Updates: A Solution to the Metadata Update Problem in File Systems<br>
	
	* Rethinking Database High Availability with RDMA Networks<br>

* consistency or (consensus)
    * Paxos Made Simple<br>
    * raft: In Search of an Understandable Consensus Algorithm <br>
    * (raft-ATC)In Search of an Understandable Consensus Algorithm<br>
    * Linearizability : A Correctness Condition for Concurrent Object<br>
    * Sequential consistency versus linearizability<br>
    * Strong and Efficient Consistency with Consistency-Aware Durability (FAST 2020 best paper)<br>
该篇文章是FAST 2020 best paper，主要讲述了在分布式存储系统中durable数据时，如何在保证strong consistency的前提下，实现higher throughput 和 lower latency，在传统的强一致性模型下，由于每一次数据修改操作都需要immediately将数据从master同步到follower上，这个过程在分布式场景下，会严重降低系统性能。本文作者观察到：在强一直模型下，如果能够保证一个cross-client 单调读的特性，那么我们就可以采用乐观的方式进行写后的读操作。本文的的方法我认为叫做optimistic read会更加的贴切，当我某个出follower中的数据时，会和durable index比较，就能够得出是否当前该数据版本是最新的，如果不是，那么系统就会在此时锁定将数据从主节点复制过来并且完成持久化，并返回结果给客户端；如果是最新的，那么直接返回给客户端结果。乍看来，这样子系统的性能岂不是会很差？但是其实不是，因为就像乐观并发控制协议一样，作者假设大部分情况下分布式系统的后台进程会将数据从master持久到replicas中，并且在存在follower的情况下，你请求到replica，并且这个数据恰好不是最新的可能性不大，**除非系统在频繁更新数据，并且也在频繁读取更新过的数据，这样性能的性能会发生抖动**。本人还解决了存在网络分区情况下的数据的读取，如果存在网络分区，那么系统会原子地保证分区节点从active set中剔除出去，这样在分区节点上的读操作就会被锁定，不允许从这个节点读取数据，等待分区情况解决之后，会重新持久最新的数据到分区节点上，并将其加入到active set中，保证单调读特性。本文的实现很优美，我觉得是durable和consistency实现中的一股清流。

* lock-free data structure
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


* query processing or query plan
    * Encapsulation of Parallehsm in the Volcano Query Procesing System <br>
    * LEO: DB2’s LEarning Optimizer <br>
	* Neo: A Learned Query Optimizer<br>
    
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
* Sigmod 2020 papers https://dl.acm.org/doi/proceedings/10.1145/3403425
* crowdsourcing https://research.yandex.com/tutorials/crowd/kdd-2019
