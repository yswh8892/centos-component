
# 原理和工作模式

```
集群方案比较:

redis3.0版本的集群模式
哨兵模式：
在redis3.0以前的版本要实现集群一般是借助哨兵sentinel工具来监控master节点的状态，如果master节点异常，则会做主从切换，将某一台slave作为master，哨兵的配置略微复杂，并且性能和高可用性等各方面表现一般，特别是在主从切换的瞬间存在访问瞬断的情况，而且哨兵模式只有一个主节点对外提供服务，没法支持很高的并发，且单个主节点内存也不宜设置得过大，否则会导致持久化文件过大，影响数据恢复或主从同步的效率。



高可用集群模式
redis集群是一个由多个主从节点群组成的分布式服务器群，它具有复制、高可用和分片特性。Redis集群不需要sentinel哨兵也能完成节点移除和故障转移的功能。需要将每个节点设置成集群模式，这种集群模式没有中心节点，可水平扩展，据官方文档称可以线性扩展到上万个节点(官方推荐不超过1000个节点)。redis集群的性能和高可用性均优于之前版本的哨兵模式，且集群配置非常简单。

```
# 准备工作

准备好三台虚拟机：
node1 （192.168.1.150）、 
node2（ 192.168.1.160）、
node3（ 192.168.1.170）

# 安装所需工具
```
wget http://download.redis.io/releases/redis-5.0.4.tar.gz
yum -y install gcc gcc-c++ libstdc++-devel tcl -y
```


# 编译和安装

```
cd /opt/redis-cluster/redis-5.0.4


make

make PREFIX=opt/redis-cluster install

安装完后，在/opt/redis-cluster中增加 bin目录

```

# 创建所需目录
```
mkdir -p /var/log/redis
mkdir -p /data1/redis/7001
mkdir -p /data1/redis/7002
mkdir -p /opt/redis-cluster/conf/7001 /opt/redis-cluster/conf/7002

```


# 需要修改的配置

```

bind 192.168.1.150 127.0.0.1
port 7001
timeout 60
daemonize yes #后台运行
protected-mode no  #（需要不同服务器的节点连通，这个就要设置为 no）
pidfile /var/run/redis_7001.pid
loglevel warning
logfile "/var/log/redis/redis_7001.log"
databases 255
dbfilename 7001dump.rdb
dir /data1/redis/7001
appendonly yes
auto-aof-rewrite-min-size 1GB
cluster-enabled yes
cluster-config-file nodes-7001.conf
cluster-node-timeout 5000


分发到其他2台机器
```

# 启动

```
  在3台机器上分别启动
  
  /opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/7001/redis.conf
  /opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/7002/redis.conf
```

# 创建集群

```
  /opt/redis-cluster/bin/redis-cli --cluster create 192.168.3.3:7001 192.168.3.4:7001 192.168.3.6:7001 192.168.3.3:7002 192.168.3.4:7002 192.168.3.6:7002 --cluster-replicas 1
  
```

# 集群运维相关

```

/opt/redis-cluster/bin/redis-cli -h 192.168.3.3 -p 7001 -c  登录

/opt/redis-cluster/bin/redis-cli --cluster check 192.168.3.3:7001 集群完整性检查



[root@k8s-master 7002]# /opt/redis-cluster/bin/redis-cli -h 192.168.1.150 -p 7001 -c  #集群信息
192.168.1.150:7001>  
192.168.1.150:7001> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:8
cluster_my_epoch:7
cluster_stats_messages_ping_sent:1528
cluster_stats_messages_pong_sent:1598
cluster_stats_messages_sent:3126
cluster_stats_messages_ping_received:1598
cluster_stats_messages_pong_received:1528
cluster_stats_messages_received:3126



192.168.1.150:7001> cluster nodes #节点信息
a5d4cf467e1e61c0e6977858dae332029474c529 192.168.1.170:7001@17001 slave dc8335f6151dbc7881d55d2cc5dc0e0b0986eebc 0 1556505376000 8 connected
c8cf578148cb6ba9094643603ae625ec0c47bbd2 192.168.1.170:7002@17002 slave fe5995113f38b2823f2fa30b276def95f56b81d0 0 1556505375000 6 connected
7dac85f2bcab4c819023e7b3cc945969af283a81 192.168.1.160:7002@17002 master - 0 1556505375000 7 connected 0-5460
feb8f7115a5a143903d75a13a2d657bf92fc500a 192.168.1.150:7001@17001 myself,slave 7dac85f2bcab4c819023e7b3cc945969af283a81 0 1556505376000 1 connected
dc8335f6151dbc7881d55d2cc5dc0e0b0986eebc 192.168.1.150:7002@17002 master - 0 1556505376518 8 connected 10923-16383
fe5995113f38b2823f2fa30b276def95f56b81d0 192.168.1.160:7001@17001 master - 0 1556505375716 2 connected 5461-10922



/opt/redis-cluster/bin/redis-cli -c -h 192.168.1.150 -p 7001 shutdown  --关闭集群，需一个个关闭


```







