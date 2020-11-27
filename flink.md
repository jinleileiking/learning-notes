# k8s

## 不能指定image

要加'' !!! wtf!

nodeselector 和 label是有参数的。仔细看doc


```
./bin/kubernetes-session.sh \
  -Dkubernetes.namespace=flink \
  -Dkubernetes.jobmanager.service-account=flink \
  -Dkubernetes.cluster-id=flink \
  -Dtaskmanager.memory.process.size=8192m \
  -Dkubernetes.taskmanager.cpu=1 \
  -Dtaskmanager.numberOfTaskSlots=4 \
  -Dresourcemanager.taskmanager-timeout=3600000 -kubernetes.jobmanager.node-selector='zone:xxxxxx' -kubernetes.taskmanager.node-selector='zone:xxx' -Dkubernetes.container.image='xxx/flink-1.11.1-scala_2.12'
  ```

## 启动命令

```
./bin/flink run -d -t kubernetes-session -Dkubernetes.cluster-id=flink -Dkubernetes.namespace=flink -Dkubernetes.jobmanager.service-account=flink   examples/streaming/WindowJoin.jar
```

```
./bin/kubernetes-session.sh \
  -Dkubernetes.namespace=flink \
  -Dkubernetes.jobmanager.service-account=flink \
  -Dkubernetes.cluster-id=flink \
  -Dtaskmanager.memory.process.size=8192m \
  -Dkubernetes.taskmanager.cpu=1 \
  -Dtaskmanager.numberOfTaskSlots=4 \
  -Dresourcemanager.taskmanager-timeout=3600000 -Dkubernetes.jobmanager.node-selector='zone:xxxxxxx' -Dkubernetes.taskmanager.node-selector='zone:xxxxx' -Dkubernetes.container.image='xsssss/public/flink-1.11.1-scala_2.12' -Dkubernetes.rest-service.exposed.type='NodePort'
```




## 8081端口连不上

`Caused by: java.util.concurrent.CompletionException: org.apache.flink.shaded.netty4.io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: /10.120.x.x:8081`

默认是loadbalance， clustip会报错，用nodeport，好使，默认使用了master的nodeport。。。。。。


# General

## kafka连不上

试了很久，启用新的consumergroup会好些，universal好像不好使，而且文档有bug： pom应该是0.11 写成011了。最后 universal  好使了，是group offset 坏了估计


## 出现direct memory 溢出

根据提示加了`taskmanager.memory.task.off-heap.size: 500m`. 加大了就不行，500m好使。


## FlinkKafkaProducer011.Semantic.EXACTLY_ONCE 不好使

去掉就行了，估计版本问题。



## middlemanager的peon报错 s3 aws 连不上

去了s3的extention就不报错了。


##  报1host=xxxxxxx 的错误

看了半天，原来是yaml `|-`  应该为 `|`  `-`的含义是删掉最后回车，operator会加点东西，直接加到末尾了。。。。

## middlemanager 会挂掉

1. 用物理机的zk，不用k8s
2. 给middlemanager 内存太少，kdp 看了一下原因，看到了oom，要不得查半天。




## flink run 找不到class

./bin/flink run --class k2h.K2h  ~/lk/k2h/target/k2h-0.1.jar


## loader violation

`Caused by: java.lang.LinkageError: loader constraint violation: loader (instance of org/apache/flink/util/ChildFirstClassLoader) previously initiated loading for a different type with name "org/apache/kafka/clients/producer/ProducerReco
rd"` 在 flink.yaml 改 loader为parent....  https://stackoverflow.com/questions/63559514/flink-fails-to-load-producerrecord-class-with-linkageerror-at-runtime


## jvm 不够

Total process memory (taskmanager.memory.process.size) 设置大


## datanode ip不对

```
2020-11-25 10:09:54,659 INFO  org.apache.hadoop.hdfs.DFSClient                             [] - Exception in createBlockOutputStream
org.apache.hadoop.net.ConnectTimeoutException: 60000 millis timeout while waiting for channel to be ready for connect. ch : java.nio.channels.SocketChannel[connection-pending remote=/x.x.x.x:50010]
```

网络不通，默认找的是内网的ip,需要 hdfs-site.xml:

```
    <property>
        <name>dfs.client.use.datanode.hostname</name>
        <value>true</value>
    </property>
```

然后添加datanode 信息到hosts


## NameNode 进入safe

查了一下是磁盘满了，hdfs-audit.log没rotate：改hdfs-log4j:

```
log4j.appender.DRFAAUDIT.MaxFileSize=10MB
log4j.appender.DRFAAUDIT.MaxBackupIndex=10
```



# flink-hive


## jar找不到

* `java.lang.NoClassDefFoundError: org/apache/flink/table/catalog/hive/HiveCatalog` 解法：           
下载相应的connector`curl -O https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-hive-2.3.6_2.11/1.11.2/flink-sql-connector-hive-2.3.6_2.11-
1.11.2.jar`             
* `java.lang.NoClassDefFoundError: org/apache/hadoop/conf/Configuration` copy hadoop-common
* `java.lang.NoClassDefFoundError: org/apache/commons/logging/LogFactory`: commons-logging
* `java.lang.NoClassDefFoundError: org/apache/hadoop/mapred/JobConf`: hadoop-mapreduce-client-core.jar
* `Caused by: java.lang.NoClassDefFoundError: Could not initialize class org.apache.hadoop.security.UserGroupInformation`   hadoop-auth.jar commons-configuration.jar--- 这个必须有，要不等会还会报。。  
* `Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.conf.Configuration`  hadoop-common.jar  
* `Caused by: java.lang.ClassNotFoundException: javax.servlet.Filter`  servlet-api.jar
* `java.lang.NoClassDefFoundError: org/apache/commons/logging/LogFactory` 解法：首先要把hadoop-client里面的jar包copy到flink/lib. 并且还要删除commons-cli.jar， 好像是因为hadoop 2.7.3里面的是1.2而在app的pom使用commons-cli.jar 1.3.1  否则会`Exception in thread "main" java.lang.NoSuchMethodError:org.apache.commons.cli.Option.builder(Ljava/lang/String;)Lorg/apache/commons/cli/Option$Builder;`


## 金山云的gateway hive不好使

`export  HADOOP_USER_NAME=hdfs`


## hive-site.conf

需要copy gateway的 /etc/hive/2.6.1.0-129/0



## Caused by: org.apache.parquet.hadoop.MemoryManager$1: New Memory allocation 1034931 bytes is smaller than the minimum allocation size of 1048576 bytes.



## create table失败
`Caused by: MetaException(message:java.security.AccessControlException: Permission denied: user=xxxx, access=WRITE, inode="/apps/hive/warehouse/xxxxxxxxx.db":hdfs:hdfs:drwxr-xr-x`  
在 client  ：` export HADOOP_USER_NAME=hdfs`


## flink run 报错

`Caused by: org.apache.kafka.common.config.ConfigException: Invalid value org.apache.flink.kafka.shaded.org.apache.kafka.common.serialization.ByteArraySerializer for configuration key.serializer: Class org.apache.flink.kafka.shaded.org.a
pache.kafka.common.serialization.ByteArraySerializer could not be found.`

之前把hadoop一堆jar放到了flink/lib，重新清了，就好了。不知道为什么。。。过了一会又碰到这个问题，是启动的flink server一定要干净，否则就不行。

## 任务上去 hdfs不行

<dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>${org.apache.hadoop.version}</version>
</dependency>

这个pom好像不管用，需要把hadoop-hdfs.jar copy到 flink/lib, 并重启flink....


## steamtable不对

按照官网用blink的，网上都是老代码

## 报没有streamgraph

这个不用env.Execute()


##  java.lang.RuntimeException: java.io.IOException: No FileSystem for scheme: hdfs

```
<dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>${org.apache.hadoop.version}</version>
</dependency>
```

## Caused by: java.lang.ClassNotFoundException: org.apache.htrace.SamplerBuilder

` cp lib_b/htrace-core.jar ./lib/ ` 重启flink....

提交上去的job，找不到jar，就要在flinklib找，copy并重启。。。。


## org.apache.flink.runtime.JobException: Recovery is suppressed by NoRestartBackoffTimeStrategy

`bsEnv.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);`

## 缺少host hdfs-ha

`flink 配置加个这个env.hadoop.conf.dir: /home/jinlei1/oss/flink/flink-standalone/flink-1.11.1/conf`


# fsql

## fsql select kafka 不通


`org.apache.flink.table.api.ValidationException: Could not find any factory for identifier 'kafka' that implements 'org.apache.flink.table.factories.DynamicTableSourceFactory' in the classpath.` 仔细阅读需要copy kafka sql connector.jar 到flink/lib


## 缺jar

* `java.lang.ClassNotFoundException: org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer` copy ./flink-sql-connector-kafka_2.11-1.11.2.jar 到 flink/lib 
* `java.lang.ClassNotFoundException: org.apache.kafka.common.serialization.ByteArrayDeserializer`  copy kafka-clients
* `org.apache.flink.streaming.runtime.tasks.StreamTaskException: Cannot load user class: org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer` 这个坑爹。整了半天。scala的版本问题。。。要用flink-dist相匹配的版本，重启服务后恢复  http://apache-flink.147419.n8.nabble.com/flink-td960.html

# coding

* StreamTableEnvironment 找不到类：`import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;`


# java

## java 删除 / 

`String ret = cut.replaceAll("\\\\", "")`


## joda 转换

19/Aug/2020:19:07:48.007 +0800   

dd/MMM/yyyy:HH:mm:ss.SSS Z


