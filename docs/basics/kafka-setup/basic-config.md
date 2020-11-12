### 储存相关

#### 	Broker

​		静态参数，必须重启生效

##### 		log.dirs

​			配置多个独立磁盘，可故障转移，保证可靠性
​			通过这个配置RAID不是必选项
​			多个磁盘同时写，增加速度
​			home/kafka1,home/kafka2

##### 		log.dir

​			不需要

### ZooKeeper相关

​	协调管理并保存 Kafka 集群的所有元数据信息
​		Broker运行情况
​		已创建Topic
​		Topic的分区
​		分区的 Leader 副本

#### 	zookeeper.connect

​		配置集群
​		zk1:2181,zk2:2181
​		一个zookeeper多个kafka的做法
​			chroot 别名
​			zk1:2181,zk2:2181/kafka1
​			zk1:2181,zk2:2181/kafka2

### Broker

#### 	listeners

​		通过什么协议访问指定主机名和端口开放的 Kafka 服务
​		若干个逗号分隔的三元组，每个三元组的格式为<协议名称，主机名，端口号>
​		协议名称可自定义
​			CONTROLLER: //localhost:9092
​			必须还要指定listener.security.protocol.map参数告诉这个协议底层使用了哪种安全协议
​			listener.security.protocol.map=CONTROLLER:PLAINTEXT
​		最好全部使用主机名，即 Broker 端和 Client 端应用配置中全部填写主机名
​	advertised.listeners
​		这组监听器是 Broker 用于对外发布的
​		常见的玩法是：你的Kafka Broker机器上配置了双网卡，一块网卡用于内网访问（即我们常说的内网IP）；另一个块用于外网访问。那么你可以配置listeners为内网IP，advertised.listeners为外网IP。
​	host.name/port
​		过期的参数

### 关于 Topic 管理

#### 	auto.create.topics.enable

​		是否允许自动创建 Topic
​		建议最好设置成 false
​			防止生产程序写错名字

#### 	unclean.leader.election.enable

​		是否允许 Unclean Leader 选举
​		显式地把它设置成 false
​			避免慢的副本竞争Leader，会丢数据

#### 	auto.leader.rebalance.enable

​		是否允许定期进行 Leader 选举
​		设置false 没有收益

### 数据留存方面

#### 	log.retention.{hours|minutes|ms}

​		控制一条消息数据被保存多长时间

#### 	log.retention.bytes

​		Broker 为消息保存的总磁盘容量大小

#### 	message.max.bytes

​		控制 Broker 能够接收的最大消息大小

### Topic 级别参数

​	Topic 级别参数会覆盖全局 Broker 参数的值

#### 	Topic 级别参数的设置

##### 		创建 Topic 时进行设置

​			bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic transaction --partitions 1 --replication-factor 1 --config retention.ms=15552000000 --config max.message.bytes=5242880

##### 		修改 Topic 时设置

​			bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name transaction --alter --add-config max.message.bytes=10485760

#### 	retention.ms

​		规定了该 Topic 消息被保存的时长。默认是 7 天

#### 	retention.bytes

​		Topic 预留多大的磁盘空间

#### 	max.message.bytes



### JVM 参数

​	JVM 堆大小设置成 6GB

#### 	Java 7

​		如果 Broker 所在机器的 CPU 资源非常充裕，建议使用 CMS 收集器
​		XX:+UseCurrentMarkSweepGC
​	否则
​		使用吞吐量收集器
​		-XX:+UseParallelGC

#### 	Java 8

​		用默认的 G1 收集器
​		G1 表现得要比 CMS 出色，主要体现在更少的 Full GC
​	KAFKA_HEAP_OPTS
​		指定堆大小
​	KAFKA_JVM_PERFORMANCE_OPTS
​		指定 GC 参数
​	启动 Kafka Broker 之前
​		export KAFKA_HEAP_OPTS=--Xms6g --Xmx6g
​		export KAFKA_JVM_PERFORMANCE_OPTS= -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true$> bin/kafka-server-start.sh config/server.properties

### 操作系统参数

#### 	文件描述符限制

​		ulimit -n 1000000
​			“Too many open files”的错误

#### 	文件系统类型

​		XFS 的性能要强于 ext4

#### 	Swappiness

​		一旦设置成 0，当物理内存耗尽时，操作系统会触发 OOM killer
​		设置成一个较小的值
​			比如 1

#### 	提交时间

​		Flush 落盘时间
​			默认是 5 秒
​		增加提交间隔来降低物理磁盘的写操作
​		Kafka 在软件层面已经提供了多副本的冗余机制
​			在页缓存中的数据在写入到磁盘前机器宕机了，那岂不是数据就丢失