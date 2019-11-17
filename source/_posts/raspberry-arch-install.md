---
title: 树莓派安装Arch Linux
tags:
  - 树莓派
categories:
  - 折腾实录
  - 树莓派
toc: true
date: 2019-11-17 22:44:15
---

因为前段时间调试树莓派拔了几次电源, 可能是有文件损坏, 直接不能开机了.
无奈只能考虑重新装下系统了, 官网的系统是带桌面的. 对树莓派来说资源有限, 桌面还是不要比较好.
于是考虑装个Arch, 一方面很干净, 另一方面社区支持很强大, 软件库很全.
以下是安装过程:
# 准备
Arch的安装需要依赖Linux环境, 所以后续步骤都需要在Linux环境中进行. 由于我物理机是用的Windows, 所以要把SD卡加载到虚拟机中.
# 虚拟机挂载SD卡:
## 查看读卡器
我是直接买的SD读卡器, 然后接到笔记本的读卡那个口的(不是USB).所以直接可移动设备这里是找不到这个SD卡的.
![image.png](/images/2019/11/17/d5f7b880-0940-11ea-b150-ff9e08d2dade.png)
如果是通过USB读卡器可以跳过此步骤, 直接在**虚拟机** -> **可移动设备**中连接设备就可以了.
正确的挂载方式:
在磁盘管理器(diskmgmt.msc)找到SD卡的编号, 如下图, 我的电脑是两个硬盘, 所以这里有磁盘0和磁盘1.如果已经插入SD卡, 会多出一个磁盘2, 确认一下磁盘2是你的SD卡, 弄错了后续操作会格式化掉选中的磁盘.
![image.png](/images/2019/11/17/45fd16c0-0941-11ea-b150-ff9e08d2dade.png)
## 添加到虚拟机
注意, 此功能需要以管理员权限运行VMware
编辑虚拟机设置, 点击**添加**
![image.png](/images/2019/11/17/004ae020-0942-11ea-b150-ff9e08d2dade.png)
在弹出的窗口选择 硬盘 -> IDE(据说SCSI也可以, 不过没测试过, 官方文档推荐的是IDE类型) -> 使用物理磁盘 -> 选择在上一步找到的设备编号, 如磁盘2, 下边的框选择整个磁盘
后续步骤填写保存的文件名即可.
# 下载ArchARM镜像
```bash
# 切换到下载路径(自定义)
cd /home/download
wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-3-latest.tar.gz
# 创建boot 和 root目录, 为安装做准备
mkdir boot root
```
# SD卡分区
将设备添加到虚机配置后, 运行虚拟机, 进入Linux环境.
本文以fdisk进行分区.
## 查看SD卡设备
```bash
lsblk
```
根据容量判断SD卡的设备名称, 如: /dev/sdb
## 分区步骤:
```bash
# 加载SD卡设备(以/dev/sdb为例, 如有差异自行修改)
fdisk /dev/sdb
# 清空已有分区
o
# 新建BOOT分区
n
# 新建主分区(p)
[ENTER]
# 选择分区1
[ENTER]
# 直接回车, 使用默认起始扇区
[ENTER]
# 将BOOT分区大小设置为200M
+200M
# 设置分区格式为fat32
t
c
# 新建根分区
n
[ENTER]
[ENTER]
[ENTER]
...一路回车(直到回到提示按m显示参数列表这里)
# 保存分区配置并退出
w
```
## 格式化&挂载分区
```bash
# 格式化boot分区
mkfs.vfat /dev/sdb1
# 格式化root分区
mkfs.ext4 /dev/sdb2
# 挂载boot分区到boot目录
mount /dev/sdb1 boot
# 挂载根分区到root目录
mount /dev/sdb2 root
```
# 安装镜像文件
## 导出镜像文件到root目录
```bash
bsdtar -xpf ArchLinuxARM-rpi-3-latest.tar.gz -C root
sync
```
如果报错: bsdtar: Error exit delayed from previous errors.升级一下bsdtar就可以了
```bash
wget https://github.com/libarchive/libarchive/archive/v3.3.2.tar.gz
tar xf v3.3.2.tar.gz
cd libarchive-3.3.2
cmake .
make -j2
```
## 复制boot目录内的文件
```bash
mv root/boot/* boot
```
## 卸载分区
```bash
umount boot root
```
# 后续操作(建议)
在完成安装步骤后, 建议不要直接拔出SD卡, 可能造成文件丢失. 因为树莓派一般是没有显示器的, 如果不能启动调试起来很麻烦, 应尽量避免此类问题.
## 关闭虚拟机
```bash
shutdown -h now
```
## 物理机移除设备
如果是用windows, 在右下角跟U盘一样移除掉SD卡设备即可.
# 完成
ArchLinuxArm版本的默认用户密码都是alarm, 主机名也是alarm.如果不知道IP地址的可以试试主机名是否能找到.

到此树莓派的安装过程就结束了, 拔掉SD卡插回树莓派通电并插上网线, 后续有时间会分享更多树莓派实用玩法, 感谢关注.