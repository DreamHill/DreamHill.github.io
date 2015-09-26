---
layout: post
title:  "在Linux上安装与配置Hadoop"
date:   2015-03-26 11:41:00
categories: 文件系统
comments: true
---

Hadoop的安装与配置

Hadoop的安装非常简单，大家可以在官网上下载到最近的几个版本，网址为http://apache.etoak.com/hadoop/core/。

Hadoop最早是为了在Linux平台上使用而开发的，但是Hadoop在UNIX、Windows和Mac OS X系统上也运行良好。不过，在Windows上运行Hadoop稍显复杂，首先必须安装Cygwin以模拟Linux环境，然后才能安装Hadoop。

在Unix上安装Hadoop的过程与在Linux上安装基本相同，因此下面不会对其进行详细介绍。

#1.在Linux上安装与配置Hadoop

在Linux上安装Hadoop之前，需要先安装两个程序：

JDK 1.6或更高版本；

SSH（安全外壳协议），推荐安装OpenSSH。

下面简述一下安装这两个程序的原因：

Hadoop是用Java开发的，Hadoop的编译及MapReduce的运行都需要使用JDK。

Hadoop需要通过SSH来启动salve列表中各台主机的守护进程，因此SSH也是必须安装的，即使是安装伪分布式版本（因为Hadoop并没有区分集群式和伪分布式）。对于伪分布式，Hadoop会采用与集群相同的处理方式，即依次序启动文件conf/slaves中记载的主机上的进程，只不过伪分布式中salve为localhost（即为自身），所以对于伪分布式Hadoop，SSH一样是必须的。

#2.安装JDK 1.6

安装JDK的过程很简单，下面以Ubuntu为例。

（1）下载和安装JDK

确保可以连接到互联网，输入命令：

    sudo apt-get install sun-java6-jdk 

输入密码，确认，然后就可以安装JDK了。

这里先解释一下sudo与apt这两个命令，sudo这个命令允许普通用户执行某些或全部需要root权限命令，它提供了详尽的日志，可以记录下每个用户使用这个命令做了些什么操作；同时sudo也提供了灵活的管理方式，可以限制用户使用命令。sudo的配置文件为/etc/sudoers。

apt的全称为the Advanced Packaging Tool，是Debian计划的一部分，是Ubuntu的软件包管理软件，通过apt安装软件无须考虑软件的依赖关系，可以直接安装所需要的软件，apt会自动下载有依赖关系的包，并按顺序安装，在Ubuntu中安装有apt的一个图形化界面程序synaptic（中文译名为“新立得”），大家如果有兴趣也可以使用这个程序来安装所需要的软件。（如果大家想了解更多，可以查看一下关于Debian计划的资料。）

（2）配置环境变量

输入命令：

    sudo gedit /etc/profile 

输入密码，打开profile文件。

在文件的最下面输入如下内容：

    #set Java Environment  
    export JAVA_HOME= （你的JDK安装位置，一般为/usr/lib/jvm/java-6-sun）  
    export CLASSPATH=".:$JAVA_HOME/lib:$CLASSPATH" 
    export PATH="$JAVA_HOME/:$PATH" 

这一步的意义是配置环境变量，使你的系统可以找到JDK。

（3）验证JDK是否安装成功

输入命令：

    java -version 

查看信息：

    java version "1.6.0_14"  
    Java(TM) SE Runtime Environment (build 1.6.0_14-b08)  
    Java HotSpot(TM) Server VM (build 14.0-b16, mixed mode) 
  
#3.配置SSH免密码登录

同样以Ubuntu为例，假设用户名为u。

1）确认已经连接上互联网，输入命令

    sudo apt-get install ssh 

2）配置为可以无密码登录本机。

首先查看在u用户下是否存在.ssh文件夹（注意ssh前面有“.”，这是一个隐藏文件夹），输入命令：

    ls -a /home/u 

一般来说，安装SSH时会自动在当前用户下创建这个隐藏文件夹，如果没有，可以手动创建一个。

接下来，输入命令：

    ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 

解释一下，ssh-keygen代表生成密钥；-t（注意区分大小写）表示指定生成的密钥类型；dsa是dsa密钥认证的意思，即密钥类型；-P用于提供密语；-f指定生成的密钥文件。（关于密钥密语的相关知识这里就不详细介绍了，里面会涉及SSH的一些知识，如果读者有兴趣，可以自行查阅资料。）

在Ubuntu中，~代表当前用户文件夹，这里即/home/u。

这个命令会在.ssh文件夹下创建两个文件id_dsa及id_dsa.pub，这是SSH的一对私钥和公钥，类似于钥匙及锁，把id_dsa.pub（公钥）追加到授权的key里面去。

输入命令：

    cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys 

这段话的意思是把公钥加到用于认证的公钥文件中，这里的authorized_keys是用于认证的公钥文件。

至此无密码登录本机已设置完毕。

3）验证SSH是否已安装成功，以及是否可以无密码登录本机。

输入命令：

    ssh -version 

显示结果：

    OpenSSH_5.1p1 Debian-6ubuntu2, OpenSSL 0.9.8g 19 Oct 2007  
    Bad escape character 'rsion'. 

显示SSH已经安装成功了。

输入命令：

    ssh localhost 

会有如下显示：

    The authenticity of host 'localhost (::1)' can't be established.  
    RSA key fingerprint is 8b:c3:51:a5:2a:31:b7:74:06:9d:62:04:4f:84:f8:77.  
    Are you sure you want to continue connecting (yes/no)? yes  
    Warning: Permanently added 'localhost' (RSA) to the list of known hosts.  
    Linux master 2.6.31-14-generic #48-Ubuntu SMP Fri Oct 16 14:04:26 UTC 2009 i686  
     
    To access official Ubuntu documentation, please visit:  
    http://help.ubuntu.com/  
     
    Last login: Mon Oct 18 17:12:40 2010 from master  
    admin@Hadoop:~$ 

这说明已经安装成功，第一次登录时会询问你是否继续链接，输入yes即可进入。

实际上，在Hadoop的安装过程中，是否无密码登录是无关紧要的，但是如果不配置无密码登录，每次启动Hadoop，都需要输入密码以登录到每台机器的DataNode上，考虑到一般的Hadoop集群动辄数百台或上千台机器，因此一般来说都会配置SSH的无密码登录。

#4.安装并运行Hadoop

介绍Hadoop的安装之前，先介绍一下Hadoop对各个节点的角色定义。

Hadoop分别从三个角度将主机划分为两种角色。第一，划分为master和slave，即主人与奴隶；第二，从HDFS的角度，将主机划分为NameNode和DataNode（在分布式文件系统中，目录的管理很重要，管理目录的就相当于主人，而NameNode就是目录管理者）；第三，从MapReduce的角度，将主机划分为JobTracker和TaskTracker（一个job经常被划分为多个task，从这个角度不难理解它们之间的关系）。

Hadoop有官方发行版与cloudera版，其中cloudera版是Hadoop的商用版本，这里先介绍Hadoop官方发行版的安装方法。

Hadoop有三种运行方式：单节点方式、单机伪分布方式与集群方式。乍看之下，前两种方式并不能体现云计算的优势，在实际应用中并没有什么意义，但是在程序的测试与调试过程中，它们还是很有意义的。

你可以通过以下地址获得Hadoop的官方发行版：

    http://www.apache.org/dyn/closer.cgi/Hadoop/core/ 

下载Hadoop-0.20.2.tar.gz并将其解压，这里会解压到用户目录下，一般为：/home/[你的用户名]/。

单节点方式配置：

安装单节点的Hadoop无须配置，在这种方式下，Hadoop被认为是一个单独的Java进程，这种方式经常用来调试。

伪分布式配置：

你可以把伪分布式的Hadoop看做是只有一个节点的集群，在这个集群中，这个节点既是master，也是slave；既是NameNode也是DataNode；既是JobTracker，也是TaskTracker。

伪分布式的配置过程也很简单，只需要修改几个文件，如下所示。

进入conf文件夹，修改配置文件：

    Hadoop-env.sh:  
    export JAVA_HOME=“你的JDK安装地址” 

指定JDK的安装位置：

    conf/core-site.xml:  
    <configuration> 
        <property> 
            <name>fs.default.name</name> 
            <value>hdfs://localhost:9000</value> 
        </property> 
    </configuration> 

这是Hadoop核心的配置文件，这里配置的是HDFS的地址和端口号。

    conf/hdfs-site.xml:  
    <configuration> 
        <property> 
            <name>dfs.replication</name> 
            <value>1</value> 
        </property> 
    </configuration> 

这是Hadoop中HDFS的配置，配置的备份方式默认为3，在单机版的Hadoop中，需要将其改为1。

    conf/mapred-site.xml:  
    <configuration> 
        <property> 
            <name>mapred.job.tracker</name> 
            <value>localhost:9001</value> 
        </property> 
    </configuration> 

这是Hadoop中MapReduce的配置文件，配置的是JobTracker的地址和端口。

需要注意的是，如果安装的是0.20之前的版本，那么只有一个配置文件，即为Hadoop-site.xml。

接下来，在启动Hadoop前，需格式化Hadoop的文件系统HDFS（这点与Windows是一样的，重新分区后的卷总是需要格式化的）。进入Hadoop文件夹，输入下面的命令：

    bin/Hadoop NameNode -format 

格式化文件系统，接下来启动Hadoop。

输入命令：

    bin/start-all.sh（全部启动） 

最后，验证Hadoop是否安装成功。

打开浏览器，分别输入网址：

    http://localhost:50030 (MapReduce的Web页面)  
    http://localhost:50070 (HDFS的Web页面) 

如果都能查看，说明Hadoop已经安装成功。

对于Hadoop来说，安装MapReduce及HDFS都是必须的，但是如果有必要，你依然可以只启动HDFS（start-dfs.sh）或MapReduce（start-mapred.sh）。

关于完全分布式的Hadoop会在2.3节详述。

#5.在Windows上安装与配置Hadoop

相对于Linux，Windows版本的JDK安装过程更容易，你可以在http://www.java.com/zh_CN/download/manual.jsp下载到最新版本的JDK。这里再次申明，Hadoop的编译及MapReduce程序的运行，很多地方都需要使用JDK的相关工具，因此只安装JRE是不够的。

安装过程十分简单，运行即可，程序会自动配置环境变量（在之前的版中还没有这项功能，新版本的JDK中已经可以自动配置环境变量了）。

2.2.1　安装Cygwin

Cygwin是在Windows平台下模拟Unix环境的一个工具，只有通过它才可以在Windows环境下安装Hadoop。可以通过这个链接下载Cygwin：

    http://www.cygwin.cn/setup.exe 

双击运行安装程序，选择install from internet。

根据网络状况，选择合适的源下载程序。

进入 select packages界面，然后进入Net，勾选openssl及openssh勾选openssl及openssh
如果打算在Eclipse上编译Hadoop，还必须安装“Base Category”下的“sed”
勾选sed
另外建议安装“Editors Category”下的“vim”，以便在Cygwin 上直接修改配置文件。

配置环境变量

依次点击我的电脑→属性→高级系统设置→环境变量，修改环境变量里的path设置，在其后添加Cygwin的bin目录和Cygwin的usr\bin目录。

#5.安装和启动sshd服务

点击桌面上的Cygwin图标，启动Cygwin，执行ssh-host-config 命令，当要求输入Yes/No时，选择输入No。当看到“Have fun”时，表示sshd 服务安装成功。

在桌面上的“我的电脑”图标上右击，点击“管理”菜单，启动CYGWIN sshd 服务。

配置SSH免密码登录

执行ssh-keygen 命令生成密钥文件。按如下命令生成authorized_keys文件：

    cd ~/..ssh/  
    cp id_rsa.pub authorized_keys 

完成上述操作后，执行exit 命令先退出Cygwin 窗口，如果不执行这一步操作，下面的操作可能会遇到错误。

接下来，重新运行Cygwin，执行ssh localhost 命令，在第一次执行时会有提示，然后输入yes，直接回车即可。

另外，在Windows上安装Hadoop的过程与Linux一样，这里就不再赘述了。

#6.安装和配置Hadoop集群

2.3.1　网络拓扑

通常来说，一个Hadoop的集群体系结构由两层网络拓扑组成，如图2-1所示。结合实际的应用来看，每个机架中会有30～40台机器，这些机器共享一个1GB带宽的网络交换机。在所有的机架之上还有一个核心交换机或路由器，通常来说其网络交换能力为1GB或更高。可以很明显地看出，同一个机架中机器节点之间的带宽资源肯定要比不同机架中机器节点间丰富。这也是Hadoop随后设计数据读写分发策略要考虑的一个重要因素。

定义集群拓扑

在实际应用中，为了使Hadoop集群获得更高的性能，读者需要配置集群使Hadoop能够感知其所在的网络拓扑结构。当然如果集群中机器数量很少，而且它们存在于一个机架中，那么就不用做太多额外的工作，而当集群中存在多个机架时，就要使Hadoop清晰地知道每台机器所在的机架。随后，在处理MapReduce任务时，Hadoop会优先选择在机架内部进行数据传输，而不是在机架间，这样就可以更充分地使用网络带宽资源。同时，HDFS可以更加智能地部署数据副本，并在性能和可靠性间寻找到最优的平衡。

在Hadoop中，网络的拓扑结构、机器节点及机架的网络位置定位都是通过树结构来描述的。通过它来确定节点间的距离，这个距离是Hadoop做决策判断时的参考因素。NameNode也是通过这个距离来决定应该把数据副本放到哪里的。当一个map任务到达时，它会被分配到一个TaskTracker上运行，JobTracker节点则会使用网络位置来确定map任务执行的机器节点。

在图2-3中，笔者使用树结构来描述网络拓扑结构，主要包括两个网络位置：交换机/机架1和交换机/机架2。因为在图中的集群只有一个最高级别的交换机，所以此网络拓扑可简化描述为/机架1和/机架2。

在配置Hadoop时，Hadoop会确定节点地址和其网络位置的映射，此映射在代码中通过Java接口DNSToSwitchMapping实现，代码如下：

    public interface DNSToSwitchMapping {  
    public List<String> resolve(List<String> names);  
    } 
    
  自己画手稿
  switch 最高
  指向 rack1、rack2 文件群负载服务器
  rack 指向诸多NODE 里的DISK
  
  其中参数names是IP地址的一个List数据，这个函数返回的值为对应网络位置的字符串列表。在opology.node.switch.mapping.impl的配置参数中定义了一个DNSToSwitchMapping接口的实现，NameNode通过它确定完成任务的机器节点所在的网络位置。

在图2-1的实例中，可以将节点1、节点2、节点3映射到/机架1中，节点4、节点5、节点6，映射到/机架2中。事实上在实际应用中，管理员可能不需要手动做额外的工作去配置这些映射关系，系统有一个默认的接口实现ScriptBasedMapping。它可以运行用户自定义的一个脚本区完成映射，如果用户没有定义，它则会将所有的机器节点映射到一个单独的网络位置中默认的机架上。如果用户定义了映射，则这个脚本的位置由topology.script.file.name的属性控制。脚本必须获取一批主机的IP地址作为参数进行映射，同时生成一个标准的网络位置给输出。

#7.建立和安装Cluster（1）

想要建立Hadoop集群，首先要做的就是选择并购买机器，在机器到手之后，就要进行网络部署并安装软件了。安装和配置Hadoop有很多方法，这一部分内容在前文已经详细讲解（见2.1节和2.2节），同时还告诉了读者在实际部署时应该考虑的情况。

为了简化我们在每个机器节点上安装和维护相同软件的过程，通常会采用自动安装法，比如Red Hat Linux下的Kickstart或Debian的全程自动化安装。这些工具先会记录你的安装过程，以及你对选择项的选择，然后根据记录来自动安装软件。同时它会在每个进程的结尾提供一个钩子执行脚本，在对那些不包含在标准安装中的最终系统进行调整和自定义时这是非常有用的。

下面我们将具体介绍一下如何部署和配置Hadoop。Hadoop为了应对不同的使用需求（不管是开发、实际应用还是研究），有着不同的运行方式，包括单节点、单机伪分布、集群等。前面已经详细介绍了Hadoop在Windows和Linux平台下的安装与配置。下面将对Hadoop的分布式配置做具体的介绍。

1. Hadoop集群的配置

在配置伪分布式的过程中，大家也许会觉得Hadoop的配置很简单，但那只是最基本的配置。

Hadoop的配置文件分为两类：

只读类型的默认文件：src/core/core-default.xml、src/hdfs/hdfs-default.xml、src/mapred/mapred-default.xml、conf/mapred-queues.xml。

定位（site-specific）设置：conf/core-site.xml、conf/hdfs-site.xml、conf/mapred-site.xml、conf/mapred-queues.xml。

除此之外，也可以通过设置conf/Hadoop-env.sh来为Hadoop的守护进程设置环境变量（在bin/文件夹内）。

Hadoop是通过org.apache.Hadoop.conf.Configuration来读取配置文件的，在Hadoop的设置中，Hadoop 的配置是通过资源（resource）定位的，每个资源由一系列name/value对以XML文件的形式构成，它以一个字符串命名或以Hadoop定义的Path类命名（这个类是用于定义文件系统中的文件或文件夹的）。如果是以字符串命名的，Hadoop会通过classpath调用此文件。如果是以Path类命名的，那么Hadoop会直接在本地文件系统中搜索文件。

资源设定有两个特点，如下所示。

Hadoop允许定义最终参数（final parameters），如果任意资源声明了final这个值，那么之后加载的任何资源都不能改变这个值，定义最终资源的格式是这样的：

    <property> 
        <name>dfs.client.buffer.dir</name> 
        <value>/tmp/Hadoop/dfs/client</value> 
        <final>true</final>   //注意这个值  
    </property> 

Hadoop允许参数传递，如下所示，当tempdir被调用时，basedir会作为值被调用。

    <property> 
        <name>basedir</name> 
        <value>/user/${user.name}</value> 
    </property> 
     
    <property> 
        <name>tempdir</name> 
        <value>${basedir}/tmp</value> 
    </property> 

刚才提到，读者可以通过设置conf/Hadoop-env.sh为Hadoop的守护进程设置环境变量。一般来说，大家至少需要在这里设置在主机上安装JDK的位置（JAVA_HOME），以使Hadoop找到JDK。大家也可以在这里通过HADOOP_*_OPTS对不同的守护进程分别进行设置，如表2-1所示。

表2-1　Hadoop的守护进程配置表
  
  Name Node  -------->Hadoop_NameNode_OPTS
  DATA Node  -------->Hadoop_DataNode_OPTS
  Secondary Name Node>Hadoop_SecondaryNameNode_OPTS
  Job Tracker-------->Hadoop_JobTracker_OPTS
  Task Tracker------->Hadoop_TaskTracker_Opts
  
例如，如果想设置NameNode使用parallelGC，那么可以这样写：

    export HADOOP_NameNode_OPTS="-XX:+UseParallelGC ${HADOOP_NameNode_OPTS}" 

在这里也可以进行其他设置，比如设置Java的运行环境（HADOOP_OPTS），设置日志文件的存放位置（HADOOP_LOG_DIR），或者SSH的配置（HADOOP_SSH_OPTS）等。

关于conf/core-site.xml、conf/hdfs-site.xml、conf/mapred-site.xml的配置可查看表2-2、表2-3和表2-4。

表2-2　conf/core-site.xml的配置

fs.default.name -------> NameNode 的IP地址及端口

表2-3　conf/hdfs-site.xml的配置


  
