#21天打卡计划 2019-12-19
#### 学习课程

《ZooKeeper实战与源码剖析》

#### 打卡笔记

第一天打卡，怎么也得跟上，先定个小目标，坚持三天 

今天把第一章基础篇整体看了一遍，主要是zookeeper的功能介绍，和基础的客户端操作实践，收货满满 。

zk是什么，可以用来干什么？
	zookeeper 开源分布式协同服务系统
	Hadoop 使用zk做namenode做高可用
	HBASE 保证集群只有一个master，同时保存集群中的regionserver列表
	kafka 集群成员管理，controller节点选举
典型应用场景
	配置管理
	DNS服务
	组成员管理
	分布式锁

zk不适用于大数据量的存储

zk的数据模型是层次模型
	文件系统的树形结构便于表达数据之间的层次关系
	文件系统的树形结构便于为不同的应用分配独立的命名空间

zk的层次模型称为daa tree。 每个节点叫做znode,每个节点都可以保存数据，且都有版本号

四种znode类型
	1. persistent
	2. persistent_sequential
	3. ephemeral
	4. ephemeral_sequential
基础客户端命令
	ls -R /  	//列出所有节点
	create -e /lock   // 创建临时节点，只有一个客户端能成功
	stat -w /lock  //监听节点，如果 /lock 节点删除会收到一个通知
使用zk实现master-worker自动切换
	create  /workers
	master1：create	-e /master "m1:8080"
	master2：create	-e /master "m2:8080"   
		master2 创建失败 ，进入 backup 状态，同时执行 stat -w /master 监听状态
		如果收到通知 立即执行 create -e /master "m2:8080"   执行成功则为master	
	master 监听workers: ls -w /workers  // 监控 /workers 目录的变化

worker1： create -e /workers/w1 "w1:8081"  
	执行成功 master能收到一个通知 
	这时master 再次执行 ls -w /workers  重新监听
	注意： master每次收到通知后，都必须重新执行 监听命令

zk的架构
	standalone 模式 。单个节点
	quorum 模式。 多个节点
	客户端和集群某一个节点创建一个session
	客户端可以主动关闭，另外在timeout时间内没有收到客户端消息，zk节点会关闭session
	客户端如果发现连接节点出错，会自动连接其他节点

quorum模式 一个 leader节点（可读可写），多个follow节点（只读），接到写请求会自动转发leader处理

数据一致性
	全局可线性化写入： 先到达leader的写请求会被先处理，leader决定写请求的执行顺序
	客户端FIFO顺序：相同客户端的请求按照发送顺序执行