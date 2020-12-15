# metastore 连不上

1. 3306端口 需要指定， 机器的默认走3307
2. 需要grant 开权限
3. cli日志在/tmp/yourname/hive.log 需要把org改为com，然后拷贝一个mysql connector到 hive/lib. 设置classpath
4. file找不到： 是因为你的客户端配置和server不一样。 干脆申请个gateway就得了


# hive client 多打日志

`hive -hiveconf hive.root.logger=DEBUG,console `
