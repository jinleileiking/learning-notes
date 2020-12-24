# metastore 连不上

1. 3306端口 需要指定， 机器的默认走3307
2. 需要grant 开权限
3. cli日志在/tmp/yourname/hive.log 需要把org改为com，然后拷贝一个mysql connector到 hive/lib. 设置classpath
4. file找不到： 是因为你的客户端配置和server不一样。 干脆申请个gateway就得了


# hive client 多打日志

`hive -hiveconf hive.root.logger=DEBUG,console `


# hive 连不上 namenode safe：

`20/12/24 15:50:43 [main]: INFO retry.RetryInvocationHandler: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.ipc.RetriableException): org.apache.hadoop.hdfs.server.namenode.SafeModeException: Cannot create directory /tmp/hive/hdfs/bd83d37c-5273-4b5a-a8e1-187067be04ec. Name node is in safe mode.
The reported blocks 79693 needs additional 39072 blocks to reach the threshold 0.9990 of total blocks 118883.
The number of live datanodes 3 has reached the minimum number 0. Safe mode will be turned off automatically once the thresholds have been reached.
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1396)` 

```
hadoop dfsadmin -safemode leave
hadoop dfsadmin -safemode get
```
