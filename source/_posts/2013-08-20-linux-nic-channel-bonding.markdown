---
layout: post
title: "Bond Linux 网卡绑定"
date: 2013-08-20 17:18
comments: true
categories: 
---

Linux中允许多个网卡作为一个网卡使用,提供负载均衡或者是备份冗余。
<!-- more-->
让我们假设我们有两个网络接口(“eth0”和“eth1”),我们把它绑定为一个(“bond0”)。

添加以下行“ /etc/modprobe.conf ”文件。

	alias bond0 bonding

先备份/etc/sysconfig/network-scripts/ifcfg-ethx

创建一个/etc/sysconfig/network-scripts/ifcfg-bond0,配置内容如下（调整网络参数为自己适用的）

	DEVICE=bond0
	BOOTPROTO=none
	ONBOOT=yes
	NETWORK=192.168.6.0
	NETMASK=255.255.255.0
	IPADDR=192.168.6.210
	USERCTL=no
	BONDING_OPTS="mode=1 miimon=100"	#可添加primary=eth0主备模式下eth0默认为活动的

修改现有的网络接口配置文件"ifcfg-eth0" 和 "ifcfg-eth1" 添加master和slave参数如下

	#eth0
	DEVICE=eth0
	MASTER=bond0
	SLAVE=yes
	USERCTL=no
	BOOTPROTO=none
	ONBOOT=yes

	#eth1
	DEVICE=eth1
	MASTER=bond0
	SLAVE=yes
	USERCTL=no
	BOOTPROTO=none
	ONBOOT=yes
	
重新启动网络服务使更改生效。
	
	# service network restart
	Shutting down interface bond0:                              [  OK  ]
	Shutting down loopback interface:                           [  OK  ]
	Bringing up loopback interface:                             [  OK  ]
	Bringing up interface bond0:  Determining if ip address 192.168.6.210 is already in use for device bond0...
	                                                            [  OK  ]
	Bringing up interface eth0:                                 [  OK  ]
	Bringing up interface eth1:                                 [  OK  ]

查看bond0网卡的信息（如下显示现在工作在eth1上面）

	# more /proc/net/bonding/bond0 
	Ethernet Channel Bonding Driver: v3.6.0 (September 26, 2009)

	Bonding Mode: fault-tolerance (active-backup)
	Primary Slave: eth0 (primary_reselect always)
	Currently Active Slave: eth1
	MII Status: up
	MII Polling Interval (ms): 100
	Up Delay (ms): 0
	Down Delay (ms): 0

	Slave Interface: eth0
	MII Status: down
	Speed: Unknown
	Duplex: Unknown
	Link Failure Count: 0
	Permanent HW addr: b0:83:fe:bf:ca:8c
	Slave queue ID: 0

	Slave Interface: eth1
	MII Status: up
	Speed: 100 Mbps
	Duplex: full
	Link Failure Count: 0
	Permanent HW addr: b0:83:fe:bf:ca:8d
	Slave queue ID: 0

使用ifconfig命令查看IP地址信息,它显示了“bond0”运行的是“eth1”目前“eth0”为备份状态。

	# fconfig
	bond0     Link encap:Ethernet  HWaddr B0:83:FE:BF:CA:8C  
	          inet addr:192.168.6.210  Bcast:192.168.47.255  Mask:255.255.255.0
                  inet6 addr: fe80::b283:feff:febf:ca8c/64 Scope:Link
                  UP BROADCAST RUNNING MASTER MULTICAST  MTU:1500  Metric:1
                  RX packets:8918 errors:0 dropped:0 overruns:0 frame:0
                  TX packets:34 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0 
	          RX bytes:924051 (902.3 KiB)  TX bytes:4006 (3.9 KiB)

	eth0      Link encap:Ethernet  HWaddr B0:83:FE:BF:CA:8C  
	          UP BROADCAST SLAVE MULTICAST  MTU:1500  Metric:1
	          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000 
	          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)
	          Interrupt:16 

	eth1      Link encap:Ethernet  HWaddr B0:83:FE:BF:CA:8C  
	          UP BROADCAST RUNNING SLAVE MULTICAST  MTU:1500  Metric:1
	          RX packets:14320 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:70 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000 
	          RX bytes:1515495 (1.4 MiB)  TX bytes:10396 (10.1 KiB)
	          Interrupt:17 

	lo        Link encap:Local Loopback  
	          inet addr:127.0.0.1  Mask:255.0.0.0
	          inet6 addr: ::1/128 Scope:Host
	          UP LOOPBACK RUNNING  MTU:65536  Metric:1
	          RX packets:24 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:24 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0 
	          RX bytes:2084 (2.0 KiB)  TX bytes:2084 (2.0 KiB)

一旦配置好bond,它就像任何其他以太网设备。例如,您可以配置别名接口来处理多个IP地址,如下所示。  
在 "/etc/sysconfig/network-scripts" 创建 "ifcfg-bond0:1" 和 "ifcfg-bond0:2" 

	# ifcfg-bond0:1 文件内容
	DEVICE=bond0:1
	BOOTPROTO=none
	ONBOOT=yes
	NETWORK=192.168.0.0
	NETMASK=255.255.255.0
	IPADDR=192.168.0.210
	USERCTL=no
	BONDING_OPTS="mode=1 miimon=100"

	# ifcfg-bond0:2 文件内容
	DEVICE=bond0:2
	BOOTPROTO=none
	ONBOOT=yes
	NETWORK=192.168.0.0
	NETMASK=255.255.255.0
	IPADDR=192.168.0.220
	USERCTL=no
	BONDING_OPTS="mode=1 miimon=100"

注意,设备名称和IP地址不同于原来的“ifcfg-bond0”文件。  
重新启动网络服务使更改生效。

	# service network restart
	Shutting down interface bond0:                             [  OK  ]
	Shutting down loopback interface:                          [  OK  ]
	Bringing up loopback interface:                            [  OK  ]
	Bringing up interface bond0:  Determining if ip address 192.168.6.210 is already in use for device bond0...
	Determining if ip address 192.168.0.210 is already in use for device bond0...
	Determining if ip address 192.168.0.220 is already in use for device bond0...
	                                                           [  OK  ]
	Bringing up interface eth0:                                [  OK  ]
	Bringing up interface eth1:                                [  OK  ]

ifconfig命令显示了三个IP地址。

	# ifconfig
	bond0     Link encap:Ethernet  HWaddr B0:83:FE:BF:CA:8C  
	          inet addr:192.168.6.210  Bcast:192.168.6.255  Mask:255.255.255.0
	          inet6 addr: fe80::b283:feff:febf:ca8c/64 Scope:Link
	          UP BROADCAST RUNNING MASTER MULTICAST  MTU:1500  Metric:1
	          RX packets:8918 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:34 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0 
	          RX bytes:924051 (902.3 KiB)  TX bytes:4006 (3.9 KiB)

	bond0:1   Link encap:Ethernet  HWaddr B0:83:FE:BF:CA:8C  
	          inet addr:192.168.0.210  Bcast:192.168.0.255  Mask:255.255.255.0
	          UP BROADCAST RUNNING MASTER MULTICAST  MTU:1500  Metric:1

	bond0:2   Link encap:Ethernet  HWaddr B0:83:FE:BF:CA:8C  
	          inet addr:192.168.0.220  Bcast:192.168.0.255  Mask:255.255.255.0
	          UP BROADCAST RUNNING MASTER MULTICAST  MTU:1500  Metric:1

	eth0      Link encap:Ethernet  HWaddr B0:83:FE:BF:CA:8C  
	          UP BROADCAST SLAVE MULTICAST  MTU:1500  Metric:1
	          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000 
	          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)
	          Interrupt:16 

	eth1      Link encap:Ethernet  HWaddr B0:83:FE:BF:CA:8C  
	          UP BROADCAST RUNNING SLAVE MULTICAST  MTU:1500  Metric:1
	          RX packets:8918 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:34 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000 
	          RX bytes:924051 (902.3 KiB)  TX bytes:4006 (3.9 KiB)
	          Interrupt:17 

	lo        Link encap:Local Loopback  
	          inet addr:127.0.0.1  Mask:255.0.0.0
	          inet6 addr: ::1/128 Scope:Host
	          UP LOOPBACK RUNNING  MTU:65536  Metric:1
	          RX packets:28 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:28 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0 
	          RX bytes:2432 (2.3 KiB)  TX bytes:2432 (2.3 KiB)

**Bond中部分参数详解**

	BONDING_OPTS="mode=1 miimon=100"

其中mode=1表示主备模式，也就是只有一块网卡是active的，只提供失效保护。如果mode=0则是负载均衡模式的，所有的网卡都是active，还有其他一些模式很少用到  
miimon=100表示每100ms检查一次链路连接状态，如果不通则会切换物理网卡  
primary=eth0表示主备模式下eth0为默认的active网卡  

miimon是毫秒数，每100毫秒触发检测线路稳定性的事件。  
mode 是ifenslave的工作状态。  

一共有7种方式：  
=0： (balance-rr) Round-robin policy: （平衡抡循环策略）：传输数据包顺序是依次传输，直到最后一个传输完毕， 此模式提供负载平衡和容错能力。  
=1： (active-backup) Active-backup policy:（主-备份策略）：只有一个设备处于活动状态。 一个宕掉另一个马上由备份转换为主设备。mac地址是外部可见得。 此模式提供了容错能力。   
=2：(balance-xor) XOR policy:（平衡 策略）： 传输根据原地址布尔值选择传输设备。 此模式提供负载平衡和容错能力。  
=3：(broadcast) broadcast policy:  （广播策略）：将所有数据包传输给所有接口。 此模式提供了容错能力。  
=4：(802.3ad) IEEE 802.3ad Dynamic link aggregation.   IEEE 802.3ad 动态链接聚合：创建共享相同的速度和双工设置的聚合组。  
=5：(balance-tlb) Adaptive transmit load balancing（适配器传输负载均衡）:没有特殊策略，第一个设备传不通就用另一个设备接管第一个设备正在处理的mac地址，帮助上一个传。  
=6：(balance-alb) Adaptive load balancing: （适配器传输负载均衡）：大致意思是包括mode5，bonding驱动程序截获 ARP 在本地系统发送出的请求，用其中之一的硬件地址覆盖从属设备的原地址。就像是在服务器上不同的人使用不同的硬件地址一样。  

这些选项可以用命令：# modinfo bonding 来查看


