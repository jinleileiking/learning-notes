# 不能指定image

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


# 8081端口连不上

`Caused by: java.util.concurrent.CompletionException: org.apache.flink.shaded.netty4.io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: /10.120.x.x:8081`

默认是loadbalance， clustip会报错，用nodeport，好使，默认使用了master的nodeport。。。。。。


# kafka连不上

试了很久，启用新的consumergroup会好些，universal好像不好使，而且文档有bug： pom应该是0.11 写成011了。最后 universal  好使了，是group offset 坏了估计


# 出现direct memory 溢出

根据提示加了`taskmanager.memory.task.off-heap.size: 500m`. 加大了就不行，500m好使。


# FlinkKafkaProducer011.Semantic.EXACTLY_ONCE 不好使

去掉就行了，估计版本问题。


# java 删除 / 

`String ret = cut.replaceAll("\\\\", "")`


# joda 转换

19/Aug/2020:19:07:48.007 +0800   

dd/MMM/yyyy:HH:mm:ss.SSS Z



# middlemanager的peon报错 s3 aws 连不上

去了s3的extention就不报错了。


# 报1host=xxxxxxx 的错误

看了半天，原来是yaml `|-`  应该为 `|`  `-`的含义是删掉最后回车，operator会加点东西，直接加到末尾了。。。。

# middlemanager 会挂掉

1. 用物理机的zk，不用k8s
2. 给middlemanager 内存太少，kdp 看了一下原因，看到了oom，要不得查半天。

# 启动命令

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
