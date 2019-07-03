---
title: Redis集群搭建完整攻略
tags:
  - Redis
  - 缓存
  - 基础服务
originContent: ''
categories:
  - 技术
  - 缓存
toc: true
date: 2019-07-02 10:00:34
---

### 1.Redis安装和配置
```bash
# 先安装编译环境
yum groupinstall "Development tools"

cd /home/packages

# 下载redis
curl -O http://download.redis.io/releases/redis-5.0.0.tar.gz

# 解压
tar -zxf redis-*.tar.gz

# 将解压的源码目录放到自己的应用目录
mv -f redis-5.0.0 /usr/local/redis

cd /usr/local/redis

# 编译安装, 编译问题可以自行搜索
./configure
make
make install

# 测试是否安装成功, 如果出现版本号说明已经安装成功
redis-cli -v
```

### 2.修改配置
```bash
vi redis.conf
```
>> bind 192.168.1.100<br>
>> port 6379<br>
>> daemonize yes   # 是否需要以守护进程启动(如果为no, 启动后在前台, ctrl-c退出进程, 可以使用nohub的方式转入后台)<br>
>> pidfile /var/run/redis_6379.pid  # 可以自行选择目录, 但是确保已经建立好指定的目录再启动服务, 否则启动报错<br>
>> logfile "/var/redis/log/6379.log" # 日志目录, 确保存在否则启动报错<br>
>> dir /var/redis/data # 数据存放目录(RDB和AOF共用), 确保存在否则启动报错<br>
>> dbfilename 6379.rdb # rdb数据持久化文件<br>
>> appendonly yes # 打开aof<br>
>> appendfilename "6379.aof" # aof数据持久化文件<br>
>> cluster-enabled yes # 集群配置, 开启时单实例机无法执行数据操作(get | set 等)<br>
>> cluster-config-file nodes-6379.conf # 自动生成, 保存到上面指定的data目录
其他配置保持默认即可.
####启动服务推荐使用redis/utils/redis_init_script
为启动脚本建立软连接方便使用
```bash
ln -s $PWD/utils/redis_init_script /usr/bin/redis

# 这个脚本默认读取的配置文件位置为 /etc/redis/$PORT.conf, 所以先在这个建立位置一个redis.conf的软连接,文件名为端口号
mkdir /etc/redis
ln -s $PWD/redis.conf /etc/redis/6379.conf

# 使用方式如下
redis [start | stop]

# 要查看服务是否已经成功开启, 可根据端口查看或使用客户端连接一次
netstat -npl | grep 6379 # 部分系统版本需要安装net-tools
```

以上步骤顺利完成后, 就可以开始集群配置了, redis集群至少要求3主3从, 所以至少需要6个单实例服务, 可以通过单机6个配置的方式, 或多个宿主机的方式配置.
### 3.集群配置
>单机多配置
>> 只需要拷贝多份配置文件和启动脚本, 将这两个文件中的端口对应的值替换为不同的端口<br>
>> 特别需要注意的是单机配置多个redis时, 时刻记得-->改端口,改端口,改端口<-- 不能两个服务重复使用同一个端口
```bash
mkdir ../redis2

cp redis.conf ../redis2/

cp utils/redis_init_script ../redis2

cd ../redis2

sed -i s/6379/7379/ redis.conf

sed -i s/6379/7379 redis_init_script

ln -s $PWD/redis.conf /etc/redis/7379.conf

./redis_init_script start

# 查看服务是否成功启动, 如果没有出现进程, 查看日志排查问题(一般是指定的文件夹不存在)
netstat -npl | grep 7379

# 如法炮制其他4个吧, 配置好后6个实例都启动起来

```
> 多机配置<br>
> 每个主机上运行一个redis其实没什么好写的,就是步骤1 和 2, 安装完成后开启服务就好了
### 4.安装ruby环境
redis对ruby的版本有要求, 目前CentOS7的仓库版本还是很低, 需要到官网下载安装
```bash
# 如果当前系统已经附带了一个低版本的ruby 需要先卸载
yum erase ruby
# 如果是之前自己使用yum安装的, 完整回滚安装操作(remove或者erase不会卸载依赖型)
yum history list ruby
# 结果如下
#    ID | 命令行                   | 日期和时间       | 操作           | 变更数
#-------------------------------------------------------------------------------
#    25 | install ruby -y          | 2018-10-04 16:14 | Install        |    8

# 拿到ID后回滚之前的操作
yum history undo 25

cd /home/packages

curl -O https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.3.tar.gz

tar -zxf ruby-2.5.3.tar.gz

cd ruby-2.5.3

./configure --disable-install-doc # 我是没安装文档, 如果需要的可以去掉后边的参数

make && make install

# 检查是否已经成功安装ruby
ruby -v

# 安装gem
cd ..

curl -O https://rubygems.org/rubygems/rubygems-2.7.7.zip

unzip rubygems-2.7.7.zip

cd rubygems-2.7.7

ruby setup.rb

gem -v

# 为ruby安装redis组件, 这一步可能比较慢
gem install redis

```
至此, ruby环境也装好了, 可以开始建集群了
### 5. 创建集群
```bash
redis-trib create --replicas 1 192.168.1.100:6379 192.168.1.100:7379 192.168.1.100:8379 192.168.1.100:9379 192.168.1.100:10379 192.168.1.100:11379

# 如果一切正常, 输出应该是长这样
# >>> Creating cluster
# >>> Performing hash slots allocation on 6 nodes...
# Using 3 masters:
# 192.168.25.110:6379
# 192.168.25.120:6379
# 192.168.25.210:6379
# Adding replica 192.168.25.120:7379 to 192.168.25.110:6379
# Adding replica 192.168.25.110:7379 to 192.168.25.120:6379
# Adding replica 192.168.25.220:6379 to 192.168.25.210:6379
# M: ed9332ea65afb70c615936bb69f3c0081e9ab472 192.168.25.110:6379
#    slots:0-5460 (5461 slots) master
# S: 281dc2be56b2dbe5c726c656b2b4b0017d396394 192.168.25.110:7379
#    replicates acae299367e8f5a742980ac6aa089baf5eb6b5a2
# M: acae299367e8f5a742980ac6aa089baf5eb6b5a2 192.168.25.120:6379
#    slots:5461-10922 (5462 slots) master
# S: caf5a016f022b6f33cd3317ef7dd643ce800131a 192.168.25.120:7379
#    replicates ed9332ea65afb70c615936bb69f3c0081e9ab472
# M: a0c41601d30159940c2bcccce08c3fad2406dfd0 192.168.25.210:6379
#    slots:10923-16383 (5461 slots) master
# S: 387442c1fbdad2a2f2c6f17506c3ebc4c01d55bc 192.168.25.220:6379
#    replicates a0c41601d30159940c2bcccce08c3fad2406dfd0
# Can I set the above configuration? (type 'yes' to accept): yes
# >>> Nodes configuration updated
# >>> Assign a different config epoch to each node
# >>> Sending CLUSTER MEET messages to join the cluster
# Waiting for the cluster to join.....
# >>> Performing Cluster Check (using node 192.168.25.110:6379)
# M: ed9332ea65afb70c615936bb69f3c0081e9ab472 192.168.25.110:6379
#    slots:0-5460 (5461 slots) master
#    1 additional replica(s)
# M: a0c41601d30159940c2bcccce08c3fad2406dfd0 192.168.25.210:6379
#    slots:10923-16383 (5461 slots) master
#    1 additional replica(s)
# M: acae299367e8f5a742980ac6aa089baf5eb6b5a2 192.168.25.120:6379
#    slots:5461-10922 (5462 slots) master
#    1 additional replica(s)
# S: 387442c1fbdad2a2f2c6f17506c3ebc4c01d55bc 192.168.25.220:6379
#    slots: (0 slots) slave
#    replicates a0c41601d30159940c2bcccce08c3fad2406dfd0
# S: 281dc2be56b2dbe5c726c656b2b4b0017d396394 192.168.25.110:7379
#    slots: (0 slots) slave
#    replicates acae299367e8f5a742980ac6aa089baf5eb6b5a2
# S: caf5a016f022b6f33cd3317ef7dd643ce800131a 192.168.25.120:7379
#    slots: (0 slots) slave
#    replicates ed9332ea65afb70c615936bb69f3c0081e9ab472
# [OK] All nodes agree about slots configuration.
# >>> Check for open slots...
# >>> Check slots coverage...
# [OK] All 16384 slots covered.
```

### 6.问题解决
    #### 6.1 问题1
> 创建集群出现如下错误<br>
>> rubygems/core_ext/kernel_require.rb:59:in `require': cannot load such file -- redis (LoadError)<br>
>
> 这种还没开始执行创建就已经脚本错误的情况, 一般是没为ruby安装redis组件引起的, 参考 第4步

    #### 6.2 问题2
> 创建集群的时候出现如下错误<br>
>> Can I set the above configuration? (type 'yes' to accept): yes
   /usr/local/lib/ruby/gems/2.4.0/gems/redis-4.0.2/lib/redis/client.rb:119:in `call': ERR Slot 0 is already busy (Redis::CommandError)
>
> 也就是提示槽xxx繁忙的时候, 将建立集群的每个节点清除数据并重置
```bash
[root@leader1 redis]# redis-cli -h leader1 6379
leader1:6379> flushall
OK
leader1:6379> cluster reset
OK
leader1:6379> quit

# 别的节点执行一样的操作
```

### 7.使用集群
#### 使用客户端连接时, 添加一个参数 -c
```bash
[root@leader1 redis2]# redis-cli -h 192.168.1.100 -p 6379 -c
# 测试集群
leader1:6379> set name sealin
-> Redirected to slot [5798] located at 192.168.25.120:6379
OK
192.168.25.120:6379>
# 可以发现操作已经被重定向到了集群中的另一个节点
# 切换一个主机获取数据试试
[root@leader1 redis2]# redis-cli -h follower1 -c
follower1:6379> get name
-> Redirected to slot [5798] located at 192.168.25.120:6379
"sealin"
192.168.25.120:6379>
# 同样的转向到了该节点获取数据

# 查看集群信息
leader1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:18037
cluster_stats_messages_pong_sent:15989
cluster_stats_messages_fail_sent:4
cluster_stats_messages_sent:34030
cluster_stats_messages_ping_received:15984
cluster_stats_messages_pong_received:15745
cluster_stats_messages_meet_received:5
cluster_stats_messages_fail_received:3
cluster_stats_messages_received:31737

# 查看节点信息
leader1:6379> cluster nodes
a0c41601d30159940c2bcccce08c3fad2406dfd0 192.168.25.210:6379@16379 master - 0 1539966870000 3 connected 10923-16383
acae299367e8f5a742980ac6aa089baf5eb6b5a2 192.168.25.120:6379@16379 master - 0 1539966869719 2 connected 5461-10922
387442c1fbdad2a2f2c6f17506c3ebc4c01d55bc 192.168.25.220:6379@16379 slave a0c41601d30159940c2bcccce08c3fad2406dfd0 0 1539966871737 4 connected
281dc2be56b2dbe5c726c656b2b4b0017d396394 192.168.25.110:7379@17379 slave acae299367e8f5a742980ac6aa089baf5eb6b5a2 0 1539966868710 5 connected
caf5a016f022b6f33cd3317ef7dd643ce800131a 192.168.25.120:7379@17379 slave ed9332ea65afb70c615936bb69f3c0081e9ab472 0 1539966870729 4 connected
ed9332ea65afb70c615936bb69f3c0081e9ab472 192.168.25.110:6379@16379 myself,master - 0 1539966863000 1 connected 0-5460
```
整个搭建流程到此结束了, 祝君好运.