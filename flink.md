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
