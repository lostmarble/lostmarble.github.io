---
layout: post
title: Azure挂载系统磁盘转移数据 
category: 技术 
keywords: nginx,sticky
---

Azure上centos6.5由于系统盘被数据写满无法登录，需要将该盘上的数据备份下来，思路是将该磁盘挂载到别的操作系统中，转移数据。

1.在web界面的仪表盘将原虚拟机删除，保留其磁盘。
2.在另外一台正在运行的Linux虚拟机，通过Azure Web管理界面“附加磁盘”。
3.登录到正在运行的Linux，查看刚刚被附加的磁盘设备，并将其挂载
```
sudo grep SCSI /var/log/messages
mount /dev/sdc1 ~/workspace/
```
4.在workspace目录下就可以找到原系统中的数据了。

注意：
1. 挂载的时候 将整个设备/dev/sdc挂载了 导致无法识别正确的文件系统。需要改成/dev/sdc1即该设备上的一个分区

2. 挂载系统盘时，虚拟机重启后无法登录，推测，由于系统盘是也可启动盘，重启时虚拟机有两个可以启动盘，无法选择导致系统问题。



