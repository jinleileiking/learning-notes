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

`Caused by: java.util.concurrent.CompletionException: org.apache.flink.shaded.netty4.io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: /10.120.x.x:8081```

默认是loadbalance， clustip会报错，用nodeport，好使，默认使用了master的nodeport。。。。。。


# kafka连不上

试了很久，启用新的consumergroup会好些，universal好像不好使，而且文档有bug： pom应该是0.11 写成011了。



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
