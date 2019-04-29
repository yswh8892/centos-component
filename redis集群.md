
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
```





