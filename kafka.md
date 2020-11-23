# kafka.common.InconsistentClusterIdException: The Cluster ID kg34qdAcRIWxF77tuPLZ_w doesn’t match stored clusterId Some(RiWfOLDaR2KmNYeern_1ow) in meta.properties

删除data目录。topic不一样了 


# kafka 在 zk的信息


https://cwiki.apache.org/confluence/display/KAFKA/Kafka+data+structures+in+Zookeeper


# [2020-11-20 11:04:02,954] WARN [Controller id=38, targetBrokerId=38] Connection to node 38 (/xx,x,x,,x:9092) could not be established. Broker may not be a
vailable. (org.apache.kafka.clients.NetworkClient)

这个是 advertised.listeners 这个配置注释掉，就好了，不知道啥原因。。。   用 0.0， ip ，都不行


# Producer clientId=producer-53] Error while fetching metadata with correlation id 6687 : {lmss-parsed=LEADER_NOT_AVAILABLE

查了半天。 这个和delete topic没反应是一样的：

1. advertiseport 要和listen port一致，最开始搞的不一致，是错的
2. host.name,  advertised.host.name 要写成外网ip，一个都不能少,  adevertised listeners 也必须写！！！！
