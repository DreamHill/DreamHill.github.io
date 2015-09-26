---
layout: post
title:  "MooseFS技术实战"
date:   2015-03-22 11:59:00
categories: 文件系统
comments: true
---
MooseFS(Moose File System，mfs)是一种分布式文件系统，它将数据分布在网络中的不同服务器上，支持FUSE，客户端可以作为一个 普通的Unix 文件系统使用MooseFS。

MooseFS中共有四种角色：主控服务器master server、元数据日志服务器metalogger server、存储块服务器chunkserver、客户端client。

主控服务器负责各个存储块服务器的管理、文件读写调度、文件空间回收以及恢复、多节点拷贝。

元数据日志服务器负责备份主控服务器的元数据、变化日志文件，文件类型为changelog_ml.*.mfs，以便在主控服务器出问题的时候可以恢复。

存储块服务器负责提供存储空间，并为客户提供数据传输。

客户端则通过fuse挂接主控服务器上所管理的存储块服务器，可以像操作本地文件一样操作MooseFS中的文件。

Keepalived是一款服务器监控软件，可以监控服务器运行状态，当服务器死机或出现故障时，可以自动将服务切换到后备服务器上。

一、配置思路

1、自1.6.5之后，MooseFS提供了metalogger服务，默认每24小时自动获得主控服务器的所有元数据和更改日志，可以作为备份主控

2、使用keepalived，监控主控服务器运行情况，当主控服务器宕掉之后，自动启动备份主控服务器，接替主控服务器

3、主控服务器与备份主控使用相同的虚拟IP提供对外服务

4、客户端通过fuse，直接访问虚拟IP提供的在存储块服务器共享的资源

二、服务器信息

10.1.1.105 主控服务器

10.1.1.104 存储块服务器、元数据日志服务器，作为备份主控

10.1.1.103 对外的提供服务的虚拟IP

10.1.1.116 存储块服务器

10.1.1.111 客户端


三、安装配置MooseFS

对于主控服务器、元数据日志服务器、存储块服务器、客户端，MooseFS提供相同的安装文件，只是根据配置参数的不同，安装不同的程序

常用的配置参数有disable-mfsmaster、disable-mfschunkserver、disable-mfsmount、disable-mfscgiserv，以及对应的enable选项，分别表示停用、启用相应的安装。所有选项默认是启用的。在安装的时候，安装程序会自动检测是否安装了fuse开发包，如果检测到，就会编译支持mfsmount(fuse 2.7.2以上版本)选项。

另外还有with-default-user、with-default-group，用于指定MooseFS所对应的用户、组

1、在http://www.moosefs.org/download.html下载软件mfs-1.6.24.tar.gz

2、主控、存储块服务器、元数据日志服务器安装

主控、存储块服务器、元数据日志服务器可以使用相同的配置安装，只需要指定安装目录、用户、组，其他的都使用默认配置

创建用户、组

[root@localhost ~]# groupadd mfs

[root@localhost ~]# useradd -g mfs mfs

解压、安装

[root@localhost ~]# tar xvf mfs-1.6.24.tar.gz

[root@localhost ~]# cd mfs-1.6.24

[root@localhost mfs-1.6.24]# ./configure --prefix=/Data/apps/mfs --with-default-user=mfs --with-default-group=mfs

[root@localhost mfs-1.6.24]# make && make install

主控服务器是MooseFS的核心，应当安装在具有高稳定性、高配置的服务器上。最关键的是内存要足够大，MooseFS对内存的要求与存放的文件个数有关。按照官方的数据，存储块服务器上的1百万文件，主控服务器需要300M内存存放相关的信息。硬盘也要大，受存储块服务器上文件、块的个数(影响元数据文件大小)、文件变更数(影响changelog)的影响;2500万文件、50小时的变更日志需要20G空间。对CPU也有较高的要求，受MooseFS中文件的操作频率影响。

在这里，10.1.1.104作为元数据日志服务器的同时，也是备份主控服务器，使用了跟主控服务器相同的配置。

存储块服务器应当提供1G以上的可用空间，才可以写入文件，实际生产环境中，至少应提供几个G的可用空间。

3、客户端安装

客户端需要安装fuse及fuse开发包，可以使用yum来安装

[root@localhost ~]# yum -y install fuse*

创建用户、组

[root@localhost ~]# groupadd mfs

[root@localhost ~]# useradd -g mfs mfs

解压、安装

[root@localhost ~]# tar xvf mfs-1.6.24.tar.gz

[root@localhost ~]# cd mfs-1.6.24

[root@localhost mfs-1.6.24]# ./configure --prefix=/Data/apps/mfs --with-default-user=mfs --with-default-group=mfs --enable-mfsmount

[root@localhost mfs-1.6.24]# make && make install

4、配置

主控服务器、存储块服务器、元数据日志服务器分别使用不同的配置文件。配置文件默认存放目录是安装目录下的etc目录，即${prefix}/etc，其中${prefix}即上面指定的/Data/apps/mfs。

主控服务器使用的配置文件是mfsmaster.cfg，可以参照${prefix}/etc下的mfsmaster.cfg.dist创建，其中注释掉的信息是当前的默认值，使用这些默认值就可以正常运行。

另外，主控服务器也用到了mfsexports.cfg文件，指定了哪些客户端机器可以远程挂载MooseFS文件系统、具有什么权限。在文件里，添加这一行

10.1.1.0/24 / rw,alldirs,maproot=0

表明10.1.1.0～10.1.1.255网段的机器都可以挂载MooseFS文件系统，具有读写、挂载任意指定的子目录权限、自动映射为root用户。

IP地址有几种表现形式：所有ip，单个ip，IP网络地址/位数掩码，IP网络地址/子网掩码，ip段范围。

权限部分中：ro 只读模式共享，rw 读写方式共享;alldirs 许挂载任何指定的子目录;maproot 映射为root或者其他的用户;password 指定客户端密码。

在默认数据目录${prefix}/var/mfs下，安装时会产生一个空的元数据文件metadata.mfs.empty，根据这个文件复制出初始的元数据文件metadata.mfs：

[root@localhost ~]# cp metadata.mfs.empty metadata.mfs

然后就可以启动master服务了

[root@localhost ~]# /Data/apps/mfs/sbin/mfsmaster start

master服务运行后，会在数据目录${prefix}/var/mfs下产生元数据备份文件metadata.mfs.back、日志文件changelog.*.mfs、sessions.mfs文件等，默认保留前50小时的日志，即mfsexports.cfg中BACK_LOGS的设置的值。

master服务每小时会把changelog.*.mfs文件合并到元数据文件中。

元数据日志服务器中使用的配置文件是mfsmetalogger.cfg，可以参照${prefix}/etc下的mfsmetalogger.cfg.dist创建，其中注释掉的信息是当前的默认值。要注意的是，其中的MASTER_HOST\MASTER_PORT指定了主控服务器的位置、端口，需要修改为正确的。默认MASTER_HOST是mfsmaster。先修改/etc/hosts文件，增加一行

　　10.1.1.103 mfsmaster

启动metalogger服务

[root@localhost ~]# /Data/apps/mfs/sbin/mfsmetalogger start

启动后，可以看到默认数据目录${prefix}/var/mfs下会复制主控服务器的元数据备份文件metadata_ml.mfs.back、日志文件changelog_ml_back.*.mfs、sessions_ml.mfs文件

另外，可以把mfsmetalogger.cfg文件中的META_DOWNLOAD_FREQ设置成1，即每小时复制一次metadata.mfs.back文件，减少恢复的延迟时间。

存储块服务器中使用的配置文件是mfschunkserver.cfg，可以参照${prefix}/etc下的mfschunkserver.cfg.dist创建，其中注释掉的信息是当前的默认值。要注意的是，其中的MASTER_HOST\MASTER_PORT指定了主控服务器的位置、端口，需要修改为正确的。默认MASTER_HOST是mfsmaster，先修改/etc/hosts文件，增加一行

10.1.1.103 mfsmaster

创建一个用于存放数据的目录，并授予权限

[root@localhost ~]# mkdir /testshared

[root@localhost ~]# chown -R mfs:mfs /testshared

另外，mfschunkserver.cfg中指定了共享硬盘使用的配置文件mfshdd.cfg，同样可以参照${prefix}/etc下的mfshdd.cfg.dist创建。在里面添加刚才配置的目录

/testshared

存储块服务器中共享的硬盘应当只供mfs使用，以便mfs能正确的管理它的自由空间。

这样就可以启动存储块服务器了。

[root@localhost ~]# /Data/apps/mfs/sbin/mfschunkserver start

客户端

修改/etc/hosts文件，增加一行

10.1.1.103 mfsmaster

创建一个作为挂载点的目录，使用mfsmount命令挂载，就可以当作本地文件夹一样操作MooseFS的目录了

[root@localhost ~]# /Data/apps/mfs/bin/mfsmount /Data/webapps/img.muzhiwan.com/mfs -H mfsmaster

其中，-H参数挂载整个mfs目录

-P 指定实际使用的端口

-S 指定挂载的子目录
四、安装配置keepalived

通过keepalived，监控主控服务器，当主控服务器10.1.1.105上的mfsmaster服务出现问题时，自动切换到元数据日志服务器10.1.1.104。

到http://www.keepalived.org/download.html 下载最新的keepalived。

解压、安装
 

    [root@localhost ~]# tar xvf keepalived-1.2.2.tar.gz  
    [root@localhost ~]# cd keepalived-1.2.2  
    [root@localhost keepalived-1.2.2]# ./configure --prefix=/  
    Keepalived configuration  
    ------------------------  
    Keepalived version       : 1.2.2  
    Compiler                 : gcc  
    Compiler flags           : -g -O2 -DETHERTYPE_IPV6=0x86dd 
    Extra Lib                : -lpopt -lssl -lcrypto   
    Use IPVS Framework       : No  
    IPVS sync daemon support : No  
    Use VRRP Framework       : Yes  
    Use Debug flags          : No 

这里只需要启用VRRP就可以

[root@localhost keepalived-1.2.2]# make && make install

安装后，配置为随机启动服务

    chmod +x /etc/rc.d/init.d/keepalived  
    chkconfig --add keepalived  
    chkconfig --level 21 keepalived on 

使用的配置文件是/etc/keepalived/keepalived.conf。

主控服务器10.1.1.105上的配置文件是，每两秒钟使用脚本检测mfsmaster运行情况，发现运行失败，就停止keepaled服务
 

    ! Configuration File for keepalived  
    global_defs {  
       router_id LVS_STTD  
    }  
    vrrp_script check_run {  
       script "/Data/apps/mfs/keepalived_check_mfsmaster.sh"  
       interval 2  
    }  
    vrrp_sync_group VG1 {  
        group {  
              VI_1  
        }  
    }  
    vrrp_instance VI_1 {  
        state MASTER  
        interface eth1  
        virtual_router_id 88  
        priority 100  
        advert_int 1  
        nopreempt  
        authentication {  
            auth_type PASS  
            auth_pass 1111  
        }  
        track_script {  
            check_run  
        }  
        virtual_ipaddress {  
            10.1.1.103  
        }  
    }  
     
    /Data/apps/mfs/keepalived_check_mfsmaster.sh脚本，如mfsmaster未运行，则停止keepalived服务  
    #!/bin/sh  
    CHECK_TIME=2 
    mfspath="/Data/apps/mfs/sbin/mfsmaster" 
    function check_mfsmaster () {  
    ps -ef | grep mfsmaster | grep "/Data/apps/mfs/sbin/mfsmaster" | grep -v "grep"  
        if [ $? = 0 ] ;then  
            MFS_OK=1 
        else  
            MFS_OK=0 
        fi  
        return $MFS_OK  
    }  
    while [ $CHECK_TIME -ne 0 ]  
    do  
            let "CHECK_TIME -= 1"  
            check_mfsmaster  
            if [ $MFS_OK = 1 ] ; then  
                    CHECK_TIME=0 
                    exit 0  
            fi  
     
            if [ $MFS_OK -eq 0 ] &&  [ $CHECK_TIME -eq 0 ] ;then  
                    /etc/init.d/keepalived stop  
                    exit 1  
            fi  
    done 

元数据日志服务器上keepalved配置
 

    ! Configuration File for keepalived  
    global_defs {  
       router_id LVS_STTD  
    }  
    vrrp_sync_group VG1 {  
        group {  
              VI_1  
        }  
    notify_master "/Data/apps/mfs/keepalived_notify.sh master"  
    notify_backup "/Data/apps/mfs/keepalived_notify.sh backup"  
    }  
    vrrp_instance VI_1 {  
        state BACKUP  
        interface eth1  
        virtual_router_id 88  
        priority 80  
        advert_int 1  
        authentication {  
            auth_type PASS  
            auth_pass 1111  
        }  
        virtual_ipaddress {  
            10.1.1.103  
        }  
    }  
    /Data/apps/mfs/keepalived_notify.sh脚本  
    #!/bin/bash  
    MFS_HOME=/Data/apps/mfs  
    MFSMARSTER=${MFS_HOME}/sbin/mfsmaster  
    MFSMETARESTORE=${MFS_HOME}/sbin/mfsmetarestore  
    MFS_DATA_PATH=${MFS_HOME}/var/mfs  
    function backup2master(){  
    $MFSMETARESTORE -m ${MFS_DATA_PATH}/metadata.mfs.back -o ${MFS_DATA_PATH}/metadata.mfs $MFS_DATA_PATH/changelog_ml*.mfs  
    $MFSMARSTER start  
    }  
    function master2backup(){  
    $MFSMARSTER stop  
    /Data/apps/mfs/sbin/mfsmetalogger start  
    }  
    function ERROR(){  
    echo "USAGE: keepalived_notify.sh master|backup "  
    }  
    case $1 in  
            master)  
            backup2master  
            ;;  
            backup)  
            master2backup  
            ;;  
            *)  
            ERROR  
            ;;  
    esac 
  五、故障切换

在10.1.1.105上停止mfsmaster服务，查看日志
 

    [root@localhost init.d]# tail -f /var/log/messages  
    May  2 12:18:40 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:64027->[114.113.149.105]  
    May  2 12:18:40 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:64027->[114.113.149.105]  
    May  2 12:18:40 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:19812->[114.113.149.105]  
    May  2 12:18:40 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:19812->[114.113.149.105]  
    May  2 12:18:41 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:9005->[114.113.149.105]  
    May  2 12:18:41 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:9005->[114.113.149.105]  
    May  2 12:18:41 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:4508->[114.113.149.105]  
    May  2 12:18:41 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:36607->[114.113.149.105]  
    May  2 12:18:41 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:31566->[114.113.149.105]  
    May  2 12:18:41 localhost snmpd[28923]: Connection from UDP: [211.157.110.180]:12040->[114.113.149.105]  
    May  2 12:19:17 localhost mfsmaster[30383]: set gid to 501  
    May  2 12:19:17 localhost mfsmaster[30383]: set uid to 501  
    May  2 12:19:17 localhost mfsmaster[28772]: matocu: closing *:9421  
    May  2 12:19:17 localhost mfsmaster[28772]: matocs: closing *:9420  
    May  2 12:19:17 localhost mfsmaster[28772]: matoml: closing *:9419  
    May  2 12:19:20 localhost Keepalived: Terminating on signal  
    May  2 12:19:20 localhost Keepalived: Stopping Keepalived v1.2.2 (04/23,2012)  
    May  2 12:19:20 localhost Keepalived_vrrp: Terminating VRRP child process on signal  
    May  2 12:19:20 localhost Keepalived_vrrp: VRRP_Instance(VI_1) removing protocol VIPs. 

查看10.1.1.104的日志
 

    [root@localhost log]# tail -f messages  
    May  2 12:19:17 localhost mfschunkserver[17620]: connection reset by Master  
    May  2 12:19:17 localhost mfsmetalogger[6105]: connection was reset by Master  
    May  2 12:19:20 localhost mfschunkserver[17620]: connecting ...  
    May  2 12:19:20 localhost mfsmetalogger[6105]: connecting ...  
    May  2 12:19:20 localhost mfschunkserver[17620]: connection failed, error: ECONNREFUSED (Connection refused)  
    May  2 12:19:20 localhost mfsmetalogger[6105]: connection failed, error: ECONNREFUSED (Connection refused)  
    May  2 12:19:23 localhost Keepalived_vrrp: VRRP_Instance(VI_1) Transition to MASTER STATE  
    May  2 12:19:23 localhost Keepalived_vrrp: VRRP_Group(VG1) Syncing instances to MASTER state  
    May  2 12:19:23 localhost mfsmaster[17690]: set gid to 504  
    May  2 12:19:23 localhost mfsmaster[17690]: set uid to 504  
    May  2 12:19:23 localhost mfsmaster[17690]: sessions have been loaded  
    May  2 12:19:23 localhost mfsmaster[17690]: exports file has been loaded  
    May  2 12:19:23 localhost mfsmaster[17690]: mfstopology configuration file (/Data/apps/mfs/etc/mfstopology.cfg) not found - network topology not defined  
    May  2 12:19:23 localhost mfsmaster[17690]: stats file has been loaded  
    May  2 12:19:23 localhost mfsmaster[17690]: master <-> metaloggers module: listen on *:9419  
    May  2 12:19:23 localhost mfsmaster[17690]: master <-> chunkservers module: listen on *:9420  
    May  2 12:19:23 localhost mfsmaster[17690]: main master server module: listen on *:9421  
    May  2 12:19:23 localhost mfsmaster[17690]: open files limit: 5000  
    May  2 12:19:23 localhost mfschunkserver[17620]: testing chunk: /Data/testshared/03/chunk_0000000000000003_00000001.mfs  
    May  2 12:19:24 localhost Keepalived_vrrp: VRRP_Instance(VI_1) Entering MASTER STATE  
    May  2 12:19:24 localhost Keepalived_vrrp: VRRP_Instance(VI_1) setting protocol VIPs.  
    May  2 12:19:24 localhost avahi-daemon[7367]: Registering new address record for 10.1.1.103 on eth1.  
    May  2 12:19:24 localhost Keepalived_vrrp: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 10.1.1.103  
    May  2 12:19:24 localhost mfschunkserver[17620]: connecting ...  
    May  2 12:19:24 localhost mfsmetalogger[6105]: connecting ...  
    May  2 12:19:25 localhost mfsmaster[17690]: chunkserver register begin (packet version: 5) - ip: 10.1.1.116, port: 9422  
    May  2 12:19:25 localhost mfsmaster[17690]: chunkserver register end (packet version: 5) - ip: 10.1.1.116, port: 9422, usedspace: 2801012736 (2.61 GiB), totalspace: 169845575680 (158.18 GiB)  
    May  2 12:19:27 localhost mfschunkserver[17620]: connected to Master  
    May  2 12:19:27 localhost mfsmetalogger[6105]: connected to Master  
    May  2 12:19:27 localhost mfsmaster[17690]: chunkserver register begin (packet version: 5) - ip: 10.1.1.104, port: 9422  
    May  2 12:19:27 localhost mfsmaster[17690]: chunkserver register end (packet version: 5) - ip: 10.1.1.104, port: 9422, usedspace: 1069522944 (1.00 GiB), totalspace: 275084394496 (256.19 GiB)  
    May  2 12:19:28 localhost mfsmetalogger[6105]: metadata downloaded 1490B/0.000284s (5.246 MB/s)  
    May  2 12:19:28 localhost mfsmetalogger[6105]: changelog_0 downloaded 0B/0.000001s (0.000 MB/s)  
    May  2 12:19:28 localhost mfsmetalogger[6105]: changelog_1 downloaded 0B/0.000001s (0.000 MB/s)  
    May  2 12:19:28 localhost mfsmetalogger[6105]: sessions downloaded 205B/0.000096s (2.135 MB/s)  
    May  2 12:19:29 localhost Keepalived_vrrp: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 10.1.1.103  
    May  2 12:19:33 localhost mfschunkserver[17620]: testing chunk: /Data/testshared/0E/chunk_000000000000000E_00000001.mfs  
    May  2 12:19:43 localhost mfschunkserver[17620]: testing chunk: /Data/testshared/0F/chunk_000000000000000F_00000001.mfs  
    May  2 12:19:53 localhost mfschunkserver[17620]: testing chunk: /Data/testshared/06/chunk_0000000000000006_00000001.mfs 

已经自动切换了

六、简单性能测试

小文件

    [root@localhost f1]# dd if=/dev/zero of=1.img bs=100K count=5000 
    5000+0 records in  
    5000+0 records out  
    512000000 bytes (512 MB) copied, 6.26102 s, 81.8 MB/s  

大文件

    [root@localhost f1]# dd if=/dev/zero of=1.img bs=1M count=5000 
    5000+0 records in  
    5000+0 records out  
    5242880000 bytes (5.2 GB) copied, 61.5205 s, 85.2 MB/s  
    [root@localhost f1]# dd if=/dev/zero of=1.img bs=50K count=5000 
    dd if=/dev/zero of=1.img bs=10K count=5000 
    5000+0 records in  
    5000+0 records out  
    256000000 bytes (256 MB) copied, 3.16866 s, 80.8 MB/s  
    [root@localhost f1]# dd if=/dev/zero of=1.img bs=10K count=5000 
    5000+0 records in  
    5000+0 records out  
    51200000 bytes (51 MB) copied, 0.582408 s, 87.9 MB/s  
    创建1000X1000个小文件  
    [root@localhost test]# time ./1000.sh  
    real 177m8.487s  
    user 6m36.276s  
    sys 32m4.413s  

本机测试

小文件

    [root@hadoop03 test]# dd if=/dev/zero of=1.img bs=100K count=5000 
    5000+0 records in  
    5000+0 records out  
    512000000 bytes (512 MB) copied, 0.871519 s, 587 MB/s  

大文件

    [root@hadoop03 test]# dd if=/dev/zero of=1.img bs=1M count=5000 
    5000+0 records in  
    5000+0 records out  
    5242880000 bytes (5.2 GB) copied, 23.7836 s, 220 MB/s  
    [root@hadoop03 test]# dd if=/dev/zero of=1.img bs=50K count=5000 
    5000+0 records in  
    5000+0 records out  
    256000000 bytes (256 MB) copied, 2.0681 s, 124 MB/s  
    1000*1000个小文件  
    [root@hadoop03 test]# time ./1000.sh  
    real 32m1.278s  
    user 5m19.947s  
    sys 28m54.985s  
    1000.sh脚本内容  
    #!/bin/bash  
    for ((i=0;i<1000;i++))  
    do  
    mkdir ${i}  
    cd ${i}  
    for ((j=0;j<1000;j++))  
     
    do  
     
    cp /Data/webapps/img.muzhiwan.com/mfs/test/1.img ${j}  
     
    done  
     
    cd ..  
     
    done  
    
七、注意

注意/etc/hosts里面mfsmaster指向虚拟ip，否则切换到从服务器时候报错
 

    Apr 24 16:19:15 localhost mfschunkserver[5833]: connecting ...  
    Apr 24 16:19:15 localhost mfsmetalogger[5829]: connecting ...  
    Apr 24 16:19:15 localhost mfschunkserver[5833]: connection failed, error: ECONNREFUSED (Connection refused)  
    Apr 24 16:19:15 localhost mfsmetalogger[5829]: connection failed, error: ECONNREFUSED (Connection refused) 

对文件夹goal的设置，会影响新增加的文件，但是不会影响已有的文件；

可以使用/Data/apps/mfs/sbin/mfscgiserv启动web gui，监控MooseFS运行情况；

也可以使用nagios监控MooseFS运行情况。

八、参考资料

http://www.moosefs.org/reference-guide.html  官方手册

http://bbs.chinaunix.net/thread-1644309-1-1.html  shinelian总结的mfs权威指南

http://sery.blog.51cto.com/10037/263515  田逸的分布式文件系统MFS(moosefs)实现存储共享(第二版)

http://blog.csdn.net/liuyunfengheda/article/details/5260278  流云随风的MFS总结

http://blog.csdn.net/pc620/article/details/6327956  常见问题
