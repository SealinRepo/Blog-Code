---
title: centos7宿主机无法与docker通信问题解决
tags:
  - 基础服务
  - Docker
  - CentOS
categories:
  - 环境配置
toc: true
date: 2020-07-10 10:14:15
---

# 问题
centos7安装docker后, 宿主机ping容器地址出现以下错误:
```bash
[root@fileserver ~]# ping 172.17.78.2
PING 172.17.78.2 (172.17.78.2) 56(84) bytes of data.
From 172.17.78.1 icmp_seq=1 Destination Host Unreachable
From 172.17.78.1 icmp_seq=2 Destination Host Unreachable
From 172.17.78.1 icmp_seq=3 Destination Host Unreachable
From 172.17.78.1 icmp_seq=4 Destination Host Unreachable
```
![image.png](/images/2020/07/10/65a9b4d4-7693-41f1-815f-9af3c88c8bb8.png)

# 解决方案1
关闭防火墙
```bash
$ sudo systemctl stop iptables
$ sudo systemctl disable iptables
$ sudo systemctl stop firewalld
$ sudo systemctl disable firewalld
```
关闭selinux
```bash
# vi /etc/selinux/config
```
将 **SELINUX=enforcing** 改为 **SELINUX=disabled**

当然也可以根据实际需要添加防火墙规则, 不过要是通过docker-compose启动容器,可能会创建docker0之外的网卡,要再次添加规则,比较麻烦.

# 解决方案2
启动时使用主机网络, 启动时加上--net=host
```bash
$ docker run -d --name test --net=host image:1.0
```
这个方案的弊端比较明显, 容器需要开放的端口不能指定, 会直接启动到宿主机的端口上, 要是有其他服务在用这个端口就启动不了容器了.

# 解决方案3
如果已经确认防火墙关闭了, 不能访问容器也不想使用主机网络模式,可以升级内核试试.
```bash
## 查看当前内核版本
# uname -r
3.10.0-327.el7.x86_64
## 导入内核源
# rpm -import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
# yum --enablerepo=elrepo-kernel install -y kernel-ml

## 安装完后, 查看内核列表
# cat /boot/grub2/grub.cfg |grep menuentry
menuentry 'CentOS Linux (5.7.8-1.el7.elrepo.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-327.el7.x86_64-advanced-3bfef048-3886-4956-9ec3-7bdbfa7f6726' {
menuentry 'CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-327.el7.x86_64-advanced-3bfef048-3886-4956-9ec3-7bdbfa7f6726' {
...

## 设置默认启动高版本内核, 注意替换版本为上面查询得到的版本号
# grub2-set-default 'CentOS Linux (5.7.8-1.el7.elrepo.x86_64) 7 (Core)'

## 重启
# reboot

## 再次查看版本
# uname -r
5.7.8-1.el7.elrepo.x86_64
```

再ping容器试试, 要是依然无法解决此问题, 关键时刻别忘了运维三法宝:
## 重启
## 重装
## 换电脑
再见~