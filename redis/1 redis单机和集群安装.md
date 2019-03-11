

## redis 单机

```shell
1、下载redis的源码包。
2、把源码包上传到linux服务器
3、解压源码包
tar -zxvf redis-3.0.0.tar.gz 
4、Make
5、make install PREFIX=/usr/local/redis


#启动 默认是前端启动模式，端口是6379
#必须要带上redis.conf否则设置的deamon启动不会生效，也就是不会读取conf文件。
cd /usr/local/redis
./redis-server redis.conf 

#连接
./redis-cli -h 127.0.0.1 -p 6379 

#关闭
./redis-cli -h 127.0.0.1 -p 6379 shutdown


#从redis的源码目录中复制redis.conf到redis的安装目录。
#修改配置文件 redis.conf
sed -i '/^#/d' redis.conf;sed -i '/^$/d' redis.conf
# 删除空行
sed -i '/^$/d' redis.conf

deamonize yes

```







## redis集群

```shell
yum install -y ruby
yum install -y rubygems

gem install redis-3.0.0.gem 



port XXXX
#bind 192.168.101.3
cluster-enabled yes 

./redis-server redis.conf 

./redis-trib.rb create --replicas 1 ...

./redis-trib.rb create 192.168.0.101:7000 192.168.0.102:7000 192.168.0.103:7000

#redis集群至少需要3个主节点，每个主节点有一个从节点总共6个节点
#replicas指定为1表示每个主节点有一个从节点
#会自动分配0-16383个slots到这三个主节点，然后repilcate到从节点。
#slots:0-5460 (5461 slots) master
#slots:5461-10922 (5462 slots) master
#slots:10923-16383 (5461 slots) master


./redis-cli -c -h 192.168.0.101 -p 7000 ，其中-c表示以集群方式连接redis，-h指定ip地址，-p指定端口号

#查询集群结点信息
cluster nodes 

#查询集群状态信息
cluster info 






###############################################################################
#添加主节点
./redis-trib.rb add-node  192.168.0.104:7000


#hash槽重新分配
#第一步 连接上集群
./redis-trib.rb reshard 192.168.101.3:7001（连接集群中任意一个可用结点都行）
#第二步：输入要分配的槽数量
#第三步：输入接收槽的结点id

#添加从节点
./redis-trib.rb add-node --slave --master-id

#删除节点
./redis-trib.rb del-node 127.0.0.1:7005 4b45eb75c8b428fbd77ab979b85080146a9bc017


```









## redis持久化方案

默认是rdb模式

rdb形式，直接保存内存里面的快照 dump.rdb 快照文件

aof形式 append only File







