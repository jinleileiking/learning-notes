# aws 开k8s

* 不要在控制台点创建，要用aws这个命令创建
* role要创建k8s-cluster-policy, k8s-policy不管用
* 子网要开两个可用区
* 安装aws命令 https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
* 配置aws https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html  key找管理员要
* `An error occurred (AccessDeniedException) when calling the DescribeCluster operation: User:  is not authorized to perform: eks:DescribeCluster on resource:    :cluster/k8s-rdqa`. 在控制台给用户添加eks权限，都点上就行，不知道是哪个
* `An error occurred (AccessDeniedException) when calling the CreateCluster operation: User: arn:aws:iam:: is not authorized to perform: eks:CreateCluster on resource: ` 给用户user添加相应的策略（ eks:CreateCluster) iam:PassRole -- 单独加不管用，选了所有就好了。
* https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html 按照这个开就可以了


# kubectl get sc

强制删除
kubectl delete pods <pod> --grace-period=0 --force


 kubectl get pods | grep redash |  awk '{print $1}' | xargs kubectl delete pod --grace-period=0 --force


# helm

redis.gloabal.XXX 不好使， 要 : : : 
 
 
# 看node上都有神马pod
 
 `kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces`

