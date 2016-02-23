---
layout: post
title: 2015-upgrade-openssh-to-v7.1-online
category: 技术 
tags: openssh
keywords: 日志分析系统
description: 在线升级openssh
---

下载openssh-7.1p1.tar.gz
http://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-7.1p1.tar.gz

备份ssh配置文件：

```
mv /etc/ssh /etc/ssh.bak
```

查看是否缺包

```
rpm -qa | egrep "gcc|make|perl|pam|pam-devel|zlib|zlib-devel"
```

如果有配置yum了的话可以直接yum安装这些包，这样既可以检验是否装了，没装的直接装上。

```
yum -y install gcc* make perl pam pam-devel zlib zlib-devel
```

先卸载完旧版本的openssh

```
rpm -e `rpm -qa | grep openssh`
```

2）编译安装新版本openssh

```
tar zxf openssh-7.1p1.tar.gz && cd openssh-7.1p1
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-pam --with-zlib --with-md5-passwords
make
make install
```

3）查看是否升级到新版本

```
ssh -V
```

4）复制启动脚本到/etc/init.d

```
cp /root/openssh-7.1p1/contrib/RedHat/sshd.init /etc/init.d/sshd
```

加入开机自启

```
 chkconfig --add sshd
 chkconfig sshd on
 chkconfig sshd --list
```

5）vim /etc/ssh/sshd_config

```
X11Forwarding yes
UseLogin yes
```

6）启动sshd，用start或reload。不要restart，restart 会直接断开连接，而并不会接着启动sshd服务，这时候要通过其他途径进入机器，然后启动sshd服务才行。

```
service sshd start
```







