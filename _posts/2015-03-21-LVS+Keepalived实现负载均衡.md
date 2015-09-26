---
layout: post
title:  "LVS+Keepalived实现负载均衡!ok"
date:   2015-03-21 10:50:55
categories: github
comments: true
---
LVS自从1998年开始，发展到现在已经是一个比较成熟的技术项目了。可以利用LVS技术实现高可伸缩的、高可用的网络服务，例如WWW服务、Cache服务、DNS服务、FTP服务、MAIL服务、视频/音频点播服务等等。
注：`(鉴于2015年时间，大家都从传统运维转移到云运维更方便，但是该掌握还是要了解一点)`

#1.LVS的体系结构

使用LVS架设的服务器集群系统有三个部分组成：

（1）最前端的负载均衡层，用Load Balancer表示；

（2）中间的服务器集群层，用Server Array表示；

（3）最底端的数据共享存储层，用Shared Storage表示；

常见负载：
域名的负载 < - > 防火硬设备负载 < － > NAT,DR级别也就是我们要介绍的 LVS <-> 容器负载

#2.LVS负载均衡机制

（1）LVS是四层负载均衡，也就是说建立在OSI模型的第四层——传输层之上，传输层上有我们熟悉的TCP/UDP，LVS支持TCP/UDP的负载均衡。因为LVS是四层负载均衡，因此它相对于其它高层负载均衡的解决办法，比如DNS域名轮流解析、应用层负载的调度、客户端的调度等，它的效率是非常高的。
（2）LVS的转发主要通过修改IP地址（NAT模式，分为源地址修改SNAT和目标地址修改DNAT）、修改目标MAC（DR模式）来实现。
①NAT模式：网络地址转换

NAT（Network Address Translation）是一种外网和内网地址映射的技术。NAT模式下，网络数据报的进出都要经过LVS的处理。LVS需要作为RS（真实服务器）的网关。当包到达LVS时，LVS做目标地址转换（DNAT），将目标IP改为RS的IP。RS接收到包以后，仿佛是客户端直接发给它的一样。RS处理完，返回响应时，源IP是RS IP，目标IP是客户端的IP。这时RS的包通过网关（LVS）中转，LVS会做源地址转换（SNAT），将包的源地址改为VIP，这样，这个包对客户端看起来就仿佛是LVS直接返回给它的。客户端无法感知到后端RS的存在。

②DR模式：直接路由

R模式下需要LVS和RS集群绑定同一个VIP（RS通过将VIP绑定在loopback实现），但与NAT的不同点在于：请求由LVS接受，由真实提供服务的服务器（RealServer, RS）直接返回给用户，返回的时候不经过LVS。详细来看，一个请求过来时，LVS只需要将网络帧的MAC地址修改为某一台RS的MAC，该包就会被转发到相应的RS处理，注意此时的源IP和目标IP都没变，LVS只是做了一下移花接木。RS收到LVS转发来的包时，链路层发现MAC是自己的，到上面的网络层，发现IP也是自己的，于是这个包被合法地接受，RS感知不到前面有LVS的存在。而当RS返回响应时，只要直接向源IP（即用户的IP）返回即可，不再经过LVS。

（3）DR负载均衡模式数据分发过程中不修改IP地址，只修改mac地址，由于实际处理请求的真实物理IP地址和数据请求目的IP地址一致，所以不需要通过负载均衡服务器进行地址转换，可将响应数据包直接返回给用户浏览器，避免负载均衡服务器网卡带宽成为瓶颈。因此，DR模式具有较好的性能，也是目前大型网站使用最广泛的一种负载均衡手段。

#3.构建实战：LVS+Keepalived实现负载均衡

（1）本次基于VMware Workstation搭建一个四台Linux（CentOS 6.4）系统所构成的一个服务器集群，其中两台负载均衡服务器（一台为主机，另一台为备机），另外两台作为真实的Web服务器（向外部提供http服务，这里仅仅使用了CentOS默认自带的http服务，没有安装其他的类似Tomcat、Jexus服务）。

（2）本次实验基于DR负载均衡模式，设置了一个VIP（Virtual IP）为192.168.80.200，用户只需要访问这个IP地址即可获得网页服务。其中，负载均衡主机为192.168.80.100，备机为192.168.80.101。Web服务器A为192.168.80.102，Web服务器B为192.168.80.103。

`请在草稿上自已画图`

以下工作针对所有服务器，也就是说要在四台服务器中都要进行配置：

（1）绑定静态IP地址

命令模式下可以执行setup命令进入设置界面配置静态IP地址；x-window界面下可以右击网络图标配置；配置完成后执行service network restart重新启动网络服务；

验证：执行命令ifconfig

（2）设定主机名

①修改当前会话中的主机名，执行命令hostname xxxx (这里xxxx为你想要改为的名字)

②修改配置文件中的主机名，执行命令vi /etc/sysconfig/network (√一般需要进行此步凑才能永久更改主机名)

验证：重启系统reboot

（3）IP地址与主机名的绑定

执行命令vi /etc/hosts,增加一行内容，如下(下面的从节点以你自己的为主，本实验搭建了两个从节点)：

192.168.80.100 lvs-master

192.168.80.101 lvs-slave

#下面是本次试验的两个真实服务器节点

192.168.80.102 lvs-webserver1

192.168.80.103 lvs-webserver2

保存后退出

验证：ping lvs-master

（4）关闭防火墙

①执行关闭防火墙命令：service iptables stop

验证：service iptables stauts

②执行关闭防火墙自动运行命令：chkconfig iptables off

验证：chkconfig --list | grep iptables

配置两台Web服务器

以下操作需要在角色为Web服务器的两台中进行，不需要在负载均衡服务器中进行操作：

（1）开启http服务

命令：service httpd start

补充：chkconfig httpd on -->将httpd设为自启动服务

（2）在宿主机访问Web网页，并通过FTP工具上传自定义网页：这里上传一个静态网页，并通过更改其中的html来区别两台Web服务器，以下图所示为例，其中一台显示from 192.168.80.102，而另一台显示from 192.168.80.103；

（3）编辑realserver脚本文件

①进入指定文件夹：cd /etc/init.d/

②编辑脚本文件：vim realserver

SNS_VIP=192.168.80.200
/etc/rc.d/init.d/functions
case "$1" in
start)
       ifconfig lo:0 $SNS_VIP netmask 255.255.255.255 broadcast $SNS_VIP
       /sbin/route add -host $SNS_VIP dev lo:0
       echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
       echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
       echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
       echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
       sysctl -p >/dev/null 2>&1
       echo "RealServer Start OK"
       ;;
stop)
       ifconfig lo:0 down
       route del $SNS_VIP >/dev/null 2>&1
       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
       echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
       echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
       echo "RealServer Stoped"
       ;;
*)
       echo "Usage: $0 {start|stop}"
       exit 1
esac
exit 0

 
这里我们设置虚拟IP为：192.168.80.200

③保存脚本文件后更改该文件权限：chmod 755 realserver

④开启realserver服务：service realserver start

配置主负载服务器

（1）安装Keepalived相关包

yum install -y keepalived

在CentOS下，通过yum install命令可以很方便地安装软件包，但是前提是你的虚拟机要联网；

（2）编辑keepalived.conf配置文件

①进入keepalived.conf所在目录：cd /etc/keepalived

②首先清除掉keepalived原有配置：> keepalived.conf

③重新编辑keepalived配置文件：vi keepalived.conf

global_defs {  
   notification_email {  
         edisonchou@hotmail.com  
   }  
   notification_email_from sns-lvs@gmail.com  
   smtp_server 192.168.80.1  
   smtp_connection_timeout 30
   router_id LVS_DEVEL  # 设置lvs的id，在一个网络内应该是唯一的
}  
vrrp_instance VI_1 {  
    state MASTER   #指定Keepalived的角色，MASTER为主，BACKUP为备          
    interface eth1  #指定Keepalived的角色，MASTER为主，BACKUP为备
    virtual_router_id 51  #虚拟路由编号，主备要一致
    priority 100  #定义优先级，数字越大，优先级越高，主DR必须大于备用DR    
    advert_int 1  #检查间隔，默认为1s
    authentication {  
        auth_type PASS  
        auth_pass 1111  
    }  
    virtual_ipaddress {  
        192.168.80.200  #定义虚拟IP(VIP)为192.168.2.33，可多设，每行一个
    }  
}  
# 定义对外提供服务的LVS的VIP以及port
virtual_server 192.168.80.200 80 {  
    delay_loop 6 # 设置健康检查时间，单位是秒                    
    lb_algo wrr # 设置负载调度的算法为wlc                   
    lb_kind DR # 设置LVS实现负载的机制，有NAT、TUN、DR三个模式   
    nat_mask 255.255.255.0                
    persistence_timeout 0          
    protocol TCP                  
    real_server 192.168.80.102 80 {  # 指定real server1的IP地址
        weight 3   # 配置节点权值，数字越大权重越高              
        TCP_CHECK {  
        connect_timeout 10         
        nb_get_retry 3  
        delay_before_retry 3  
        connect_port 80  
        }  
    }  
    real_server 192.168.80.103 80 {  # 指定real server2的IP地址
        weight 3  # 配置节点权值，数字越大权重越高  
        TCP_CHECK {  
        connect_timeout 10  
        nb_get_retry 3  
        delay_before_retry 3  
        connect_port 80  
        }  
     }  
} 

（3）开启keepalived服务

service keepalived start

 配置从负载服务器

从负载服务器与主负载服务器大致相同，只是在keepalived的配置文件中需要改以下两处：

（1）将state由MASTER改为BACKUP

（2）将priority由100改为99

vrrp_instance VI_1 {  
    state BACKUP # 这里改为BACKUP
    interface eth1  
    virtual_router_id 51  
    priority 99 # 这里改为99，master优先级是100
    advert_int 1  
    authentication {  
        auth_type PASS  
        auth_pass 1111  
    }  
    virtual_ipaddress {  
        192.168.80.200  
    }  
}  
验证性测试

（1）指定请求的均衡转发：因为两个Web服务器的权重都一样，所以会依次转发给两个Web服务器；
2）Web服务器发生故障时：

①A发生故障后，只从B获取服务；

这里模拟192.168.80.102发生故障，暂停其http服务：service httpd stop

②A故障修复后，又从A获取服务；

这里模拟192.168.80.102修复完成，重启其http服务：service httpd start

（3）主负载均衡服务器发生故障时，备机立即充当主机角色提供请求转发服务：

这里模拟192.168.80.100发生故障，暂停其keepalived服务：service keepalived stop

学习小结

LVS是目前广为采用的软件负载均衡解决方案，在一些大型企业级系统及互联网系统中应用。本次，简单地了解了一下LVS，并在Linux下搭建了一个小小的测试环境，借助Keepalived实现了一个最小化的负载均衡测试环境。LVS是一个可以工作在网络第四层的负载均衡软件，因此它相对于Nginx一类工作在第七层的负载均衡软件有着无可比拟的性能优势，而且它还是我国的章文嵩博士（现在阿里的副总裁，淘宝的技术专家）作为创始人发起的，现已经成为Linux内核的组成部分。

当然，目前流行的LVS解决方案中，在Web服务器端也有采用了Nginx+Tomcat这样的搭配类型，静态文件和动态文件分开进行处理，也不失为一种有效的尝试。在以后的日子里，我还会尝试下在Linux下借助Jexus跑ASP.NET MVC项目，试试.NET项目在Linux下的运行效果，希望到时也可以做一些分享。好了，今天就到此停笔。
