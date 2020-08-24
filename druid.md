# k8s 部署 druid

首先使用helm charts进行尝试，太不好使了，首先chart比较老，根本跑步起来，于是使用operator进行部署. 首先学习operator，然后安装operator.

最后跑的pod如下：

```
default              pod/druid-operator-85648779d7-hbhjh                          1/1     Running             0          19d
default              pod/druid-tiny-cluster-brokers-0                             1/1     Running             0          20h
default              pod/druid-tiny-cluster-coordinators-0                        1/1     Running             0          20h
default              pod/druid-tiny-cluster-historicals-0                         1/1     Running             0          20h
default              pod/druid-tiny-cluster-middlemanagers-0                      1/1     Running             0          20h
default              pod/druid-tiny-cluster-overlords-0                           1/1     Running             0          20h
default              pod/druid-tiny-cluster-routers-0                             1/1     Running             0          20h
default             service/druid-operator-metrics                          ClusterIP      10.254.203.115   <none>                8383/TCP,8686/TCP                                         19d
default             service/druid-tiny-cluster-brokers                      ClusterIP      10.254.204.50    <none>                8088/TCP                                                  20h
default             service/druid-tiny-cluster-coordinators                 ClusterIP      10.254.223.35    <none>                8088/TCP                                                  20h
default             service/druid-tiny-cluster-historicals                  ClusterIP      10.254.66.72     <none>                8088/TCP                                                  20h
default             service/druid-tiny-cluster-middlemanagers               ClusterIP      10.254.147.55    <none>                8091/TCP,8100/TCP,8101/TCP,8102/TCP,8103/TCP,8104/TCP     20h
default             service/druid-tiny-cluster-routers                      ClusterIP      10.254.136.224   <none>                8888/TCP                                                  20h
default             service/overlord-druid-tiny-cluster-overlords-service   ClusterIP      10.254.187.33    <none>                8090/TCP                                                  20h
default             deployment.apps/druid-operator                          1/1       1            1           19d
default             replicaset.apps/druid-operator-85648779d7                          1         1         1       19d
default            statefulset.apps/druid-tiny-cluster-brokers          1/1     20h
default            statefulset.apps/druid-tiny-cluster-coordinators     1/1     20h
default            statefulset.apps/druid-tiny-cluster-historicals      1/1     20h
default            statefulset.apps/druid-tiny-cluster-middlemanagers   1/1     20h
default            statefulset.apps/druid-tiny-cluster-overlords        1/1     20h
default            statefulset.apps/druid-tiny-cluster-routers          1/1     20h
```

安装记录：

1. 先在k8s部署operator的pod: 



```
* kgaa -l name=druid-operator
NAMESPACE   NAME                                  READY   STATUS    RESTARTS   AGE
default     pod/druid-operator-85648779d7-hbhjh   1/1     Running   0          19d

NAMESPACE   NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
default     service/druid-operator-metrics   ClusterIP   10.254.203.115   <none>        8383/TCP,8686/TCP   19d

NAMESPACE   NAME                                        DESIRED   CURRENT   READY   AGE
default     replicaset.apps/druid-operator-85648779d7   1         1         1       19d
```

这里遇到一个有rs，但是没有pod的问题，忘记怎么解决的了。。。。。。印象里改了改，然后重新create就好了。以后真得时刻记录

2. 安装druid：

官网的实例yaml，只是能跑，broker都没有。找了很多资料，最后凑出来一个能跑的。

路上遇到的坑：

* 新版的operator/overlord的配置挪到一个目录了，配置出错，log打不出来，改一下目录就好了
* console显示restricted mode是因为哪没通，最终都跑通了，所有的按钮就好了
* router没有ingress，需要自己建立ingress，router才能出来
* metadata如果使用derby的话，不好使，最后申请了个mysql，最终跑起来了, indexer也要改为使用metadata
* 官网没有druid-mysql镜像，按照官网步骤需要自己编一个，用dockerhub autobuild ARG传不进去，我自己编了个镜像`https://hub.docker.com/repository/docker/leiking/druid-mysql`，这个dockerfile完美的展示了，如何在一个已有的镜像里，添加东西，老外真牛
* 权限问题，我最终都改为0了，否则建不了data目录，会报错
* 我把service从None改为有ip了，当初调试时弄得，感觉不用ip也行


相关异常log
```
Error in custom provider, org.apache.druid.java.util.common.RE: server initialization exception
1) Unknown provider[localdruid.host=172.10.73.7] of Key[type=org.apache.druid.indexing.overlord.TaskStorage, annotation=[none]], known options[[local, metadata]]
```


* 有一台虚机的flannel配置错了，导致dns解析不到，运维改了，重启flannel后，最终跑起来了


# select 后只展示小时级别的数据

需要调整query为second，就行了


# 查询一定时间内推流很多的sql

注意 select必须带所有group的东西

```
SELECT "name", "unique_name", "lmds_description", FLOOR(__time to MINUTE) AS Mi, SUM("count") AS c
FROM "ss-publish1"
WHERE "__time" >= CURRENT_TIMESTAMP - INTERVAL '1' DAY AND "lmds_description" != 'Url expired!'
GROUP BY 1,2,3,4 ORDER BY c DESC
```




