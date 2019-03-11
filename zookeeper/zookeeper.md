安装和配置

```shell
mv zoo_sample.cfg zoo.cfg
```



zoo.cfg

```shell
  1 # The number of milliseconds of each tick
  2 tickTime=2000
  3 # The number of ticks that the initial 
  4 # synchronization phase can take
  5 initLimit=10
  6 # The number of ticks that can pass between 
  7 # sending a request and getting an acknowledgement
  8 syncLimit=5
  9 # the directory where the snapshot is stored.
 10 # do not use /tmp for storage, /tmp here is just 
 11 # example sakes.
 12 dataDir=/tmp/zookeeper
 13 # the port at which the clients will connect
 14 clientPort=2181
 15 # the maximum number of client connections.
 16 # increase this if you need to handle more clients
 17 #maxClientCnxns=60
 18 #
 19 # Be sure to read the maintenance section of the 
 20 # administrator guide before turning on autopurge.
 21 #
 22 # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
 23 #
 24 # The number of snapshots to retain in dataDir
 25 #autopurge.snapRetainCount=3
 26 # Purge task interval in hours
 27 # Set to "0" to disable auto purge feature
 28 #autopurge.purgeInterval=1
```



```shell

#启动
cd bin
./zkServer.sh start 

#连接到zookeeper
./zkCli.sh -server localhost:2181

发现报错
2019-03-02 03:00:33,411 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error)
2019-03-02 03:00:33,413 [myid:] - WARN  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1102] - Session 0x0 for server null, unexpected error, closing socket connection and attempting reconnect
java.net.ConnectException: Connection refused
        at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
        at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
        at org.apache.zookeeper.ClientCnxnSocketNIO.doTransport(ClientCnxnSocketNIO.java:361)
        
        没连接上，可能是没有启动成功，再启动一下就完事了。。真是醉了。。连接不上这种错误，真是日了狗。。
        
```



```shell
 help
 
 #创建2个普通节点
 create /huang aaa
 create /huang niubi 
 
 #创建短暂节点
 create -e /huang/niubi "zhouyu"
 
 #创建带序号的节点
 reate -s /huang/weiguo/xiaoqiao "jinlian"
 
 
 #设置节点的值
 set /zk_test junk
 
 #获取节点的值
 get /zk_test
 
 #注册监听/huang/niubi节点数据变化
 get /huang/niubi watch
 
 #监听/huang节点的子节点的变化
 ls /huang watch
 
 #查看节点状态
stat /huang
cZxid = 0x100000005
ctime = Sun Mar 03 16:56:06 CST 2019
mZxid = 0x100000005
mtime = Sun Mar 03 16:56:06 CST 2019
pZxid = 0x100000005
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0

1）czxid-创建节点的事务zxid
每次修改ZooKeeper状态都会收到一个zxid形式的时间戳，也就是ZooKeeper事务ID。
事务ID是ZooKeeper中所有修改总的次序。每个修改都有唯一的zxid，如果zxid1小于zxid2，那么zxid1在zxid2之前发生。
2）ctime - znode被创建的毫秒数(从1970年开始)
3）mzxid - znode最后更新的事务zxid
4）mtime - znode最后修改的毫秒数(从1970年开始)
5）pZxid-znode最后更新的子节点zxid
6）cversion - znode子节点变化号，znode子节点修改次数
7）dataversion - znode数据变化号
8）aclVersion - znode访问控制列表的变化号
9）ephemeralOwner- 如果是临时节点，这个是znode拥有者的session id。如果不是临时节点则是0。


 #删除节点
 delete /zk_test
 #递归删除节点
 rmr /huang/
```





```shell
#配置分布式集群 192.168.0.101
server.1=192.168.0.101:2888:3888
server.2=192.168.0.101:2888:3888
server.3=192.168.0.101:2888:3888
```









