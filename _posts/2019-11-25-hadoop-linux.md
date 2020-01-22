---
layout : post
title : "hadoop集群搭建"
category : hadoop
duoshuo: true
date : 2019-11-25
tags : [hadoop,大数据]
---

### Hadoop 集群搭建 ###
1. 集群环境准备
    - 1.1 使用虚拟机工具创建三个服务器节点 命名 qiping lingling huanhuan
    - 1.2 关闭防火墙
    ````
    service iptables status  -- 查看状态
    service iptables stop  -- 停止
    service iptables start -- 启动
    service iptables restart  
    chkconfig iptables off  
    chkconfig iptables on　　
    ````
    - 1.3 设置host和hostname （需重启）
    ````
    vi /etc/hosts
    192.168.56.110  qiping
    192.168.56.111  lingling
    192.168.56.112  huanhuan
    scp /etc/hosts lingling:/etc/
    scp /etc/hosts huanhuan:/etc/
    ````
    - 1.4 ssh 免密码登录 (各节点)
    ````
    ssh-keygen -t rsa
    ssh-copy-id root@qiping
    ssh-copy-id root@lingling
    ssh-copy-id root@huanhuan
    ````
    
2. 安装JDK 设置环境变量
    - 3.1 下载解压
    - 3.2 vi /etc/profile -- 环境变量 包含Zookeeper Hadoop
    ````
    export JAVA_HOME=/usr/local/app/jdk1.8.0_231
    export ZOOKEEPER_HOME=/usr/local/app/apache-zookeeper-3.5.6-bin
    export HADOOP_HOME=/usr/local/app/hadoop-3.2.1
    export PATH=$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
    ````
    - 3.3 可以直接拷贝到其他节点 
    ````
    scp /etc/profile lingling:/etc/
    scp /etc/profile huanhuan:/etc/
    ````
3. 安装Zookeeper
    - 3.1 下载 apache-zookeeper-3.5.6-bin.tar.gz 解压
    - 3.2
    ````
    mv zoo_sample.cfg zoo.cfg 
    vi zoo.cfg 
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/usr/local/app/apache-zookeeper-3.5.6-bin/data
    dataLogDir=/usr/local/app/apache-zookeeper-3.5.6-bin/dataLog
    clientPort=2181
    server.1=qiping:2888:3888
    server.2=lingling:2888:3888
    server.3=huanhuan:2888:3888
    ````
    - 3.3 启动
    ````
    zkServer.sh start
    ````
    
4. 安装Hadoop
    - 4.1 下载 hadoop-3.2.1.tar.gz 解压
    - 4.2 修改配置文件
      - 4.2.1 修改 {HADOOP_HOME}/etc/hadoop/core-site.xml 添加如下
      
      ```xml
      <configuration>
      <property>
          <name>hadoop.tmp.dir</name>
          <value>/usr/local/app/hadoop-3.2.1/tmp</value>
      </property>
      <property>
            <name>fs.defaultFS</name>
            <value>hdfs://ns</value>
       </property>
      <property>
            <name>ha.zookeeper.quorum</name>
            <value>qiping:2181,lingling:2181,lingling:2181</value>
      </property>
      </configuration>
      ```
      - 4.2.2 修改 {HADOOP_HOME}/etc/hadoop/hadoop-env.sh 添加如下
      
      ```
      export JAVA_HOME=/usr/local/app/jdk1.8.0_231
      export HDFS_NAMENODE_USER=root
      export HDFS_DATANODE_USER=root
      export HDFS_SECONDARYNAMENODE_USER=root
      ````
      - 4.2.3 修改 {HADOOP_HOME}/etc/hadoop/hdfs-site.xml 添加如下
      
      ```xml
      <configuration>
              <property>
                      <name>dfs.nameservices</name>
                      <value>ns</value>
              </property>
              <property>
                      <name>dfs.ha.namenodes.ns</name>
                      <value>nn1,nn2</value>
              </property>
              <property>
                      <name>dfs.namenode.rpc-address.ns.nn1</name>
                      <value>qiping:9000</value>
              </property>
              <property>
                      <name>dfs.namenode.http-address.ns.nn1</name>
                      <value>qiping:9870</value>
              </property>
              <property>
                      <name>dfs.namenode.rpc-address.ns.nn2</name>
                      <value>lingling:9000</value>
              </property>
              <property>
                      <name>dfs.namenode.http-address.ns.nn2</name>
                      <value>lingling:9870</value>
              </property>
              <property>
                      <name>dfs.namenode.shared.edits.dir</name>
                      <value>qjournal://qiping:8485;lingling:8485;huanhuan:8485/ns</value>
              </property>
              <property>
                      <name>dfs.journalnode.edits.dir</name>
                      <value>/usr/local/app/hadoop-3.2.1/journaldata</value>
              </property>
              <property>
                      <name>dfs.ha.automatic-failover.enabled</name>
                      <value>true</value>
              </property>
              <property>
                      <name>dfs.client.failover.proxy.provider.ns</name>
                      <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
              </property>
              <property>
                      <name>dfs.ha.fencing.methods</name>
                      <value>
                              sshfence
                              shell(/bin/true)
                      </value>
              </property>
              <property>
                      <name>dfs.ha.fencing.ssh.private-key-files</name>
                      <value>/root/.ssh/id_rsa</value>
              </property>
              <property>
                      <name>dfs.ha.fencing.ssh.connect-timeout</name>
                      <value>30000</value>
              </property>
      </configuration>
      ```
      - 4.4 修改 {HADOOP_HOME}/etc/hadoop/mapred-site.xml 添加如下
      
      ````xml
      <configuration>
      	<property>
      		<name>mapreduce.framework.name</name>
      		<value>yarn</value>
      	</property>
      	<property>
      	  <name>yarn.app.mapreduce.am.env</name>
      	  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
      	</property>
      	<property>
      	  <name>mapreduce.map.env</name>
      	  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
      	</property>
      	<property>
      	  <name>mapreduce.reduce.env</name>
      	  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
      	</property>
      </configuration>
      ````
    - 4.5 修改 {HADOOP_HOME}/etc/hadoop/yarn-site.xml 添加如下
    
    ```xml`
    <configuration>
      <property>    
        <name>yarn.nodemanager.aux-services</name>    
        <value>mapreduce_shuffle</value>    
        </property>  
        <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>huanhuan</value>
          </property>
    </configuration>
    ````
    - 4.6 修改 {HADOOP_HOME}/etc/hadoop/workers 添加如下
    ````
    qiping
    lingling
    huanhuan
    ````
    
5. 启动
    - 5.1 启动 journalnode (qiping)
    ````
    hdfs --workers --daemon start journalnode
    ````
    - 5.2 格式化 HDFS (qiping)
    ````
    hdfs namenode -format
    ````
    - 5.3 格式化ZKFC (qiping)
    ````
    hdfs zkfc -formatZK 
    ````
    - 5.4 启动NameNode (qiping lingling)
    ````
    hdfs --daemon start namenode
    ````
    - 5.5 同步元数据 (lingling)
    ````
    hdfs namenode -bootstrapStandby
    ````
    - 5.6 启动并验证DataNode (qiping)
    ````
    hdfs --workers --daemon start datanode
    ````
    - 5.7 启动并验证YARN (huanhuan)
    ````
    start-yarn.sh
    ````
    - 5.8 启动并验证ZKFC (qiping)
    ````
    hdfs --workers --daemon start zkf
    ````
    - 5.9 验证
      - 5.9.1 验证 qiping
      ````
      [root@qiping hadoop]# jps
      3366 DFSZKFailoverController
      27592 Jps
      3001 DataNode
      3145 NodeManager
      2638 JournalNode
      2814 NameNode
      2703 QuorumPeerMain
      ````
      - 5.9.2 验证lingling
      ````
      2944 DFSZKFailoverController
      2401 QuorumPeerMain
      2513 NameNode
      32485 Jps
      2327 JournalNode
      2777 NodeManager
      2651 DataNode
      ````
      - 5.9.3 验证huanhuan
      ````
      2272 JournalNode
      2834 NodeManager
      2485 DataNode
      2345 QuorumPeerMain
      24683 Jps
      2684 ResourceManager
      ````

