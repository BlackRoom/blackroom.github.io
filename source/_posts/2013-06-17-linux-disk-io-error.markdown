---
layout: post
title: "Linux Disk I/O error"
date: 2013-06-17 16:34
comments: true
categories: 
---

今天线上某一大数据存储应用挂掉了，查看日志发现大量的I/O错误，记录下。
<!--more-->

应用突然挂掉查看应用日志发现有几个表出现写入错误，接着查看系统日志，发现大量如下信息。

	kernel: end_request: I/O error, dev sda, sector 155697224
	kernel: sd 0:2:0:0: [sda]  Result: hostbyte=DID_ERROR driverbyte=DRIVER_OK
	kernel: sd 0:2:0:0: [sda] CDB: Read(10): 28 00 09 47 c0 48 00 00 08 00

第一个想到的是硬盘坏了，这台机器上面是8块600GSAS硬盘做RAID0目的是为了充分利用磁盘空间和读写速度。

**利用MegaCli检查下RAID的状态已经磁盘状况**

MegaCli是一款管理维护硬件RAID软件，可以通过它来了解当前raid卡的所有信息，包括 raid卡的型号，raid的阵列类型，raid 上各磁盘状态，等等
  

	/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL

	…………
	Bad Blocks Exist: Yes	#显示有坏块

接着查看下是那几块硬盘有问题  

	/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL

	…………
	Slot Number: 0
	Media Error Count: 20	#正常状态下为0
	Other Error Count: 0
	…………
	Slot Number: 6
	Media Error Count: 2	#正常状态下为0
	Other Error Count: 0

发现有两块硬盘中有坏块，分别是第一块和第5块。机架上的硬盘标识是从0开始的。

**利用Badblocks命令将磁盘中的坏块找出来**

badblocks是linux及其类似的操作系统中，扫描检查硬盘和外部设备损坏扇区的命令工具。损坏的扇区或者损坏的区块是硬盘中因为永久损坏或者是操作系统不能读取的空间。

	fdisk -l	#查看下系统中的分区
	badblocks -v /dev/sda2	
	badblocks -v /dev/sda2  /tmp/bad-blocks.txt	#也可以打印到文件中

这个时候应该尽早将应用下线，替换掉有坏块的硬盘，当然你也可以继续使用，建议换掉。

如果要继续使用硬盘可以使用**e2fsck**命令强迫操作系统不使用这些损坏的区块存储数据。  
利用badblocks扫描检查硬盘的结果使用e2fsck命令来配置操作系统不在这些损坏的扇区中存储数据。

	e2fsck -l /tmp/bad-blocks.txt  /dev/sda2	#使用之前扫描坏块打印到的文件

注意：在运行e2fsck命令前，请保证设备没有被挂载。




