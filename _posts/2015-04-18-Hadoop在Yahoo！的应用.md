---
layout: post
title:  "Hadoop在Yahoo！的应用"
date:   2015-04-18 21:11:00
categories: 文件系统
comments: true
---

参考： <a href="http://book.51cto.com/art/201110/298604.htm">http://book.51cto.com/art/201110/298604.htm</a>

#1.Hadoop在Yahoo！的应用

#2.Hadoop在eBay的应用

#3.Hadoop在百度的应用

<a href="http://book.51cto.com/art/201110/298609.htm">Hadoop在Facebook的应用</a>

<a href="http://book.51cto.com/art/201110/298610.htm">Hadoop平台上的海量数据排序</a>

#1.小结

随着企业的数据量的迅速增长，存储和处理大规模数据已成为企业的迫切需求。Hadoop作为开源的云计算平台，已引起了学术界和企业的普遍兴趣。

在学术方面，Hadoop得到了各科研院所的广泛关注，多所著名大学加入到Hadoop集群的研究中来，其中包括斯坦福大学、加州大学伯克利分校、康奈尔大学、卡耐基梅隆大学、普渡大学等。一些国内高校和科研院所如中科院计算所、清华大学、中国人民大学等也开始对Hadoop展开相关研究，研究内容涉及Hadoop的数据存储、资源管理、作业调度、性能优化、系统可用性和安全性等多个方面。

在商业方面，Hadoop技术已经在互联网领域得到了广泛的应用。互联网公司往往需要存储海量的数据并对其进行处理，而这正是Hadoop的强项。如Facebook使用Hadoop 存储内部的日志拷贝，以及数据挖掘和日志统计；Yahoo！利用Hadoop支持广告系统并处理网页搜索；Twitter则使用Hadoop存储微博数据、日志文件和其他中间数据等。在国内，Hadoop同样也得到了许多公司的青睐，如百度主要将Hadoop应用于日志分析和网页数据库的数据挖掘；阿里巴巴则将Hadoop用于商业数据的排序和搜索引擎的优化等。

下面我们将选取具有代表性的Hadoop应用案例进行分析，让读者了解Hadoop在企业界的应用情况。

3.1　Hadoop在Yahoo！的应用

关于Hadoop技术的研究和应用，Yahoo！始终处于领先地位，它将Hadoop应用于自己的各种产品中，包括数据分析、内容优化、反垃圾邮件系统、广告的优化选择、大数据处理和ETL等；同样，在用户兴趣预测、搜索排名、广告定位等方面得到了充分的应用。

在Yahoo！主页个性化方面，实时服务系统通过Apache从数据库中读取user到interest的映射，并且每隔5分钟生产环境中的Hadoop集群就会基于最新数据重新排列内容，每隔7分钟则在页面上更新内容。

在邮箱方面，Yahoo！利用Hadoop集群根据垃圾邮件模式为邮件计分，并且每隔几个小时就在集群上改进反垃圾邮件模型，集群系统每天还可以推动50亿次的邮件投递。

目前Hadoop最大的生产应用是Yahoo！的Search Webmap应用，它运行在超过10 000台机器的Linux系统集群里，Yahoo！的网页搜索查询使用的就是它产生的数据。Webmap的构建步骤如下：首先进行网页的爬取，同时产生包含所有已知网页和互联网站点的数据库，以及一个关于所有页面及站点的海量数据组；然后将这些数据传输给Yahoo！搜索中心执行排序算法。在整个过程中，索引中页面间的链接数量将会达到1TB，经过压缩的数据产出量会达到300TB，运行一个MapReduce任务就需使用超过10 000的内核，而在生产环境中使用数据的存储量超过5PB。

Yahoo！在Hadoop中同时使用了Hive和Pig，在许多人看来，Hive和Pig大体上相似而且Pig Latin与SQL也十分相似。那么Yahoo！为什么要同时使用这些技术呢？主要是因为Yahoo！的研究人员在查看了它们的工作负载并分析了应用案例后认为不同的情况下需要使用不同的工具。

先了解一下大规模数据的使用和处理背景。大规模的数据处理经常分为三个不同的任务：数据收集、数据准备和数据表示，这里并不打算介绍数据收集阶段，因为Pig和Hive主要用于数据准备和数据表示阶段。

数据准备阶段通常被认为是提取、转换和加载（Extract Transform Load，ETL）数据的阶段，或者认为这个阶段是数据工厂。这里的数据工厂只是一个类比，在现实生活中的工厂接收原材料后会生产出客户所需的产品，而数据工厂与之相似，它在接收原始数据后，可以输出供客户使用的数据集。这个阶段需要装载和清洗原始数据，并让它遵守特定的数据模型，还要尽可能地让它与其他数据源结合等。这一阶段的客户一般都是程序员、数据专家或研究者。

数据表示阶段一般指的都是数据仓库，数据仓库存储了客户所需要的产品，客户会根据需要选取合适的产品。这一阶段的客户可能是系统的数据工程师、分析师或决策者。

根据每个阶段负载和用户情况的不同，Yahoo！在不同的阶段使用不同的工具。结合了诸如Oozie等工作流系统的Pig特别适合于数据工厂，而Hive则适合于数据仓库。下面将分别介绍数据工厂和数据仓库。

Yahoo！的数据工厂存在三种不同的工作用途：流水线、迭代处理和科学研究。

经典的数据流水线包括数据反馈、清洗和转换。一个常见例子是Yahoo！的网络服务器日志，这些日志需要进行清洗以去除不必要的信息，数据转换则是要找到点击之后所转到的页面。Pig 是分析大规模数据集的平台，它建立在Hadoop之上并提供了良好的编程环境、优化条件和可扩展的性能。Pig Latin是关系型数据流语言，并且是Pig核心的一部分，基于以下的原因，Pig Latin相比于SQL而言，更适合构建数据流。首先，Pig Latin是面向过程的，并且Pig Latin允许流水线开发者自定义流水线中检查点的位置；其次，Pig Latin允许开发者直接选择特定的操作实现方式而不是依赖于优化器；最后，Pig Latin支持流水线的分支，并且Pig Latin允许流水线开发者在数据流水线的任何地方插入自己的代码。Pig和诸如Oozie等的工作流工具一起使用来创建流水线，一天可以运行数以万计的Pig作业。

迭代处理也是需要Pig的，在这种情况下通常需要维护一个大规模的数据集。数据集上的典型处理包括加入一小片数据后就会改变大规模数据集的状态。如考虑这样一个数据集，它存储了Yahoo！新闻中现有的所有新闻。我们可以把它想象成一幅巨大的图，每个新闻就是一个节点，新闻节点若有边相连则说明这些新闻指的是同一个事件。每隔几分钟就会有新的新闻加入进来，这些工具需要将这些新闻节点加到图中，并找到相似的新闻节点用边连接起来，还要删除被新节点覆盖的旧节点。这和标准流水线不同的是它不断有小变化，这就需要使用增长处理模型在合理的时间范围内处理这些数据了。例如，所有的新节点加入图中后，又有一批新的新闻节点到达，在整个图上重新执行连接操作是不现实的，这也许会花费数个小时。相反，在新增加的节点上执行连接操作并使用全连接（full join）的结果是可行的，而且这个过程只需要花费几分钟时间。标准的数据库操作可以使用Pig Latin通过上述方式实现，这时Pig就会得到很好的应用。

Yahoo！有许多的科研人员，他们需要用网格工具处理千万亿大小的数据，还有许多研究人员希望快速地写出脚本来测试自己的理论或获得更深的理解。但是在数据工厂中，数据不是以一种友好的、标准的方式呈现的，这时Pig就可以大显身手了，因为它可以处理未知模式的数据，还有半结构化和非结构化的数据。Pig与streaming相结合使得研究者在小规模数据集上测试的Perl和Python脚本可以很方便地在大规模数据集上运行。

在数据仓库处理阶段，有两个主要的应用：商业智能分析和特定查询（Ad-hoc query）。在第一种情况下，用户将数据连接到商业智能（BI）工具（如MicroStrategy）上来产生报告或深入的分析。在第二种情况下，用户执行数据分析师或决策者的特定查询。这两种情况下，关系模型和SQL都很好用。事实上，数据仓库已经成为SQL使用的核心，它支持多种查询并具有分析师所需的工具，Hive作为Hadoop的子项目为其提供了SQL接口和关系模型，现在Hive团队正开始将Hive与BI工具通过接口（如ODBC）结合起来使用。

Pig在Yahoo！得到了广泛应用，这使得数据工厂的数据被移植到Hadoop上运行成为可能。随着Hive的深入使用，Yahoo！打算将数据仓库移植到Hadoop上。在同一系统上部署数据工厂和数据仓库将会降低数据加载到仓库的时间，这也使得共享工厂和仓库之间的数据、管理工具、硬件等成为可能。Yahoo！在Hadoop上同时使用多种工具使Hadoop能够执行更多的数据处理。

#2.Hadoop在eBay的应用

在eBay上存储着上亿种商品的信息，而且每天有数百万种的新商品增加，因此需要用云系统来存储和处理PB级别的数据，而Hadoop则是个很好的选择。

Hadoop是建立在商业硬件上的容错、可扩展、分布式的云计算框架，eBay利用Hadoop建立了一个大规模的集群系统—Athena，它被分为五层（如图3-1所示），下面从最底层向上开始介绍：

监视和警告层   - Ganglia,Nagios
工具和加载库层 - HUE,UC4,Oozie,Mobius,Mahout
数据获取层   - HBase , Hive,Pig
mapreduce层  - Java,Streaming,Pipes,Scala
Hadoop核心层 - HDFS,common

1）Hadoop核心层，包括Hadoop运行时环境、一些通用设施和HDFS，其中文件系统为读写大块数据而做了一些优化，如将块的大小由128MB改为256MB。

2）MapReduce层，为开发和执行任务提供API和控件。

3）数据获取层，现在数据获取层的主要框架是HBase、Pig和Hive：

HBase是根据Google BigTable开发的按列存储的多维空间数据库，通过维护数据的划分和范围提供有序的数据，其数据储存在HDFS上。

Pig（Latin）是提供加载、筛选、转换、提取、聚集、连接、分组等操作的面向过程的语言，开发者使用Pig建立数据管道和数据工厂。

Hive是用于建立数据仓库的使用SQL语法的声明性语言。对于开发者、产品经理和分析师来说，SQL接口使得Hive成为很好的选择。

4）工具和加载库层，UC4是eBay从多个数据源自动加载数据的企业级调度程序。加载库有：统计库（R）、机器学习库（Mahout）、数学相关库（Hama）和eBay自己开发的用于解析网络日志的库（Mobius）。

5）监视和警告层，Ganglia是分布式集群的监视系统，Nagios则用来警告一些关键事件如服务器不可达、硬盘已满等。

eBay的企业服务器运行着64位的RedHat Linux：

NameNode负责管理HDFS的主服务器；

JobTracker负责任务的协调；

HBaseMaster负责存储HBase存储的根信息，并且方便与数据块或存取区域进行协调；

ZooKeeper是保证HBase一致性的分布式锁协调器。

用于存储和计算的节点是1U大小的运行Cent OS的机器，每台机器拥有2个四核处理器和2TB大小的存储空间，每38～42个节点单元为一个rack，这组建成了高密度网格。有关网络方面，顶层rack交换机到节点的带宽为1Gbps，rack交换机到核心交换机的带宽为40Gpbs。

这个集群是eBay内多个团队共同使用的，包括产品和一次性任务。这里使用Hadoop公平调度器（Fair Scheduler）来管理分配、定义团队的任务池、分配权限、限制每个用户和组的并行任务、设置优先权期限和延迟调度。

数据流的具体处理过程如图3-2所示，系统每天需要处理8TB至10TB的新数据，而Hadoop主要用于：

基于机器学习的排序，使用Hadoop计算需要考虑多个因素（如价格、列表格式、卖家记录、相关性）的排序函数，并需要添加新因素来验证假设的扩展功能，以增强eBay物品搜索的相关性。

对物品描述数据的挖掘，在完全无人监管的方式下使用数据挖掘和机器学习技术将物品描述清单转化为与物品相关的键/值对，以扩大分类的覆盖范围。

eBay的研究人员在系统构建和使用过程中遇到的挑战及一些初步计划有以下几个方面：

可扩展性，当前主系统的NameNode拥有扩展的功能，随着集群的文件系统不断增长，需要存储大量的元数据，所以内存占有量也在不断增长。若是1PB的存储量则需要将近1GB的内存量，可能的解决方案是使用等级结构的命名空间划分，或者使用HBase和ZooKeeper联合对元数据进行管理。

有效性，NameNode的有效性对产品的工作负载很重要，开源社区提出了一些备用选择，如使用检查点和备份节点、从Secondary NameNode中转移到Avatar节点、日志元数据复制技术等。eBay研究人员根据这些方法建立了自己的产品集群。

数据挖掘，在存储非结构化数据的系统上建立支持数据管理、数据挖掘和模式管理的系统。新的计划提议将Hive的元数据和Owl添加到新系统中，并称为Howl。eBay研究人员努力将这个系统联系到分析平台上去，这样用户可以很容易地在不同的数据系统中挖掘数据。

数据移动，eBay研究人员考虑发布数据转移工具，这个工具可以支持在不同的子系统如数据仓库和HDFS之间进行数据的复制。

策略，通过配额实现较好的归档、备份等策略（Hadoop现有版本的配额需要改进）。eBay的研究人员基于工作负载和集群的特点对不同的集群确定配额。

标准，eBay研究人员开发健壮的工具来为数据来源、消耗情况、预算情况、使用情况等进行度量。

同时eBay正在改变收集、转换、使用数据的方式，以提供更好的商业智能服务。

#3.Hadoop在百度的应用

百度作为全球最大的中文搜索引擎公司，提供基于搜索引擎的各种产品，包括以网络搜索为主的功能性搜索；以贴吧为主的社区搜索；针对区域、行业的垂直搜索、MP3音乐搜索，以及百科等，几乎覆盖了中文网络世界中所有的搜索需求。

百度对海量数据处理的要求是比较高的，要在线下对数据进行分析，还要在规定的时间内处理完并反馈到平台上。百度在互联网领域的平台需求如图3-3所示，这里就需要通过性能较好的云平台进行处理了，Hadoop就是很好的选择。在百度，Hadoop主要应用于以下几个方面：

日志的存储和统计；

网页数据的分析和挖掘；

商业分析，如用户的行为和广告关注度等；

在线数据的反馈，及时得到在线广告的点击情况；

用户网页的聚类，分析用户的推荐度及用户之间的关联度。

MapReduce主要是一种思想，不能解决所有领域内与计算有关的问题，百度的研究人员认为比较好的模型应该如图3-4所示，HDFS实现共享存储，一些计算使用MapReduce解决，一些计算使用MPI解决，而还有一些计算需要通过两者来共同处理。因为MapReduce适合处理数据很大且适合划分的数据，所以在处理这类数据时就可以用MapReduce做一些过滤，得到基本的向量矩阵，然后通过MPI进一步处理后返回结果，只有整合技术才能更好地解决问题。

商业分析、网页处理机   -> 数据分析平台  ->    分布试存储平台

贴吧、空间   ->  查询平台    ->   分布式存储平台

百度现在拥有3个Hadoop集群，总规模在700台机器左右，其中有100多台新机器和600多台要淘汰的机器（它们的计算能力相当于200多台新机器），不过其规模还在不断的增加中。现在每天运行的MapReduce任务在3000个左右，处理数据约120TB/天。

百度为了更好地用Hadoop进行数据处理，在以下几个方面做了改进和调整：

（1）调整MapReduce策略

限制作业处于运行状态的任务数；

调整预测执行策略，控制预测执行量，一些任务不需要预测执行；

根据节点内存状况进行调度；

平衡中间结果输出，通过压缩处理减少I/O负担。 

用户  ->  MapReduce  ->  HDFS

用户  ->  MPI    -> HDFS

2）改进HDFS的效率和功能

权限控制，在PB级数据量的集群上数据应该是共享的，这样分析起来比较容易，但是需要对权限进行限制；

让分区与节点独立，这样，一个分区坏掉后节点上的其他分区还可以正常使用；

修改DFSClient选取块副本位置的策略，增加功能使DFSClient选取块时跳过出错的DataNode；

解决VFS（Virtual File System）的POSIX（Portable Operating System Interface of Unix）兼容性问题。

（3）修改Speculative的执行策略

采用速率倒数替代速率，防止数据分布不均时经常不能启动预测执行情况的发生；

增加任务时必须达到某个百分比后才能启动预测执行的限制，解决reduce运行等待map数据的时间问题；

只有一个map或reduce时，可以直接启动预测执行。

（4）对资源使用进行控制

对应用物理内存进行控制。如果内存使用过多会导致操作系统跳过一些任务，百度通过修改Linux内核对进程使用的物理内存进行独立的限制，超过阈值可以终止进程。

分组调度计算资源，实现存储共享、计算独立，在Hadoop中运行的进程是不可抢占的。

在大块文件系统中，X86平台下一个页的大小是4KB。如果页较小，管理的数据就会很多，会增加数据操作的代价并影响计算效率，因此需要增加页的大小。

百度在使用Hadoop时也遇到了一些问题，主要有：

MapReduce的效率问题：比如，如何在shuffle效率方面减少I/O次数以提高并行效率；如何在排序效率方面设置排序为可配置的，因为排序过程会浪费很多的计算资源，而一些情况下是不需要排序的。

HDFS的效率和可靠性问题：如何提高随机访问效率，以及数据写入的实时性问题，如果Hadoop每写一条日志就在HDFS上存储一次，效率会很低。

内存使用的问题：reducer端的shuffle会频繁地使用内存，这里采用类似Linux的buddy system来解决，保证Hadoop用最小的开销达到最高的利用率；当Java 进程内容使用内存较多时，可以调整垃圾回收（GC）策略；有时存在大量的内存复制现象，这会消耗大量CPU资源，同时还会导致内存使用峰值极高，这时需要减少内存的复制。

作业调度的问题：如何限制任务的map和reduce计算单元的数量，以确保重要计算可以有足够的计算单元；如何对TaskTracker进行分组控制，以限制作业执行的机器，同时还可以在用户提交任务时确定执行的分组并对分组进行认证。

性能提升的问题：UserLogs cleanup在每次task结束的时候都要查看一下日志，以决定是否清除，这会占用一定的任务资源，可以通过将清理线程从子Java进程移到TaskTracker来解决；子Java进程会对文本行进行切割而map和reduce进程则会重新切割，这将造成重复处理，这时需要关掉Java进程的切割功能；在排序的时候也可以实现并行排序来提升性能；实现对数据的异步读写也可以提升性能。

健壮性的问题：需要对mapper和reducer程序的内存消耗进行限制，这就要修改Linux内核，增加其限制进程的物理内存的功能；也可以通过多个map程序共享一块内存，以一定的代价减少对物理内存的使用；还可以将DataNode和TaskTracker的UGI配置为普通用户并设置账号密码；或者让DataNode和TaskTracker分账号启动，确保HDFS数据的安全性，防止Tracker操作DataNode中的内容；在不能保证用户的每个程序都很健壮的情况下，有时需要将进程终止掉，但要保证父进程终止后子进程也被终止。

Streaming局限性的问题：比如，只能处理文本数据，mapper和reducer按照文本行的协议通信，无法对二进制的数据进行简单处理。为了解决这个问题，百度人员新写了一个类Bistreaming（Binary Streaming），这里的子Java进程mapper和reducer按照（KeyLen，Key，ValLen，Value）的方式通信，用户可以按照这个协议编写程序。

用户认证的问题：这个问题的解决办法是让用户名、密码、所属组都在NameNode和Job Tracker上集中维护，用户连接时需要提供用户名和密码，从而保证数据的安全性。

百度下一步的工作重点可能主要会涉及以下内容：

内存方面，降低NameNode的内存使用并研究JVM的内存管理；

调度方面，改进任务可以被抢占的情况，同时开发出自己的基于Capacity的作业调度器，让等待作业队列具有优先级且队列中的作业可以设置Capacity，并可以支持TaskTracker分组；

压缩算法，选择较好的方法提高压缩比、减少存储容量，同时选取高效率的算法以进行shuffle数据的压缩和解压；

对mapper程序和reducer程序使用的资源进行控制，防止过度消耗资源导致机器死机。以前是通过修改Linux内核来进行控制的，现在考虑通过在Linux中引入cgroup来对mapper和reducer使用的资源进行控制；

将DataNode的并发数据读写方式由多线程改为select方式，以支持大规模并发读写和Hypertable的应用。

百度同时也在使用Hypertable，它是以Google发布的BigTable为基础的开源分布式数据存储系统，百度将它作为分析用户行为的平台，同时在元数据集中化、内存占用优化、集群安全停机、故障自动恢复等方面做了一些改进。

