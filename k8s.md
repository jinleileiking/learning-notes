# aws 开k8s

* 不要在控制台点创建，要用aws这个命令创建
* role要创建k8s-cluster-policy, k8s-policy不管用
* 子网要开两个可用区
* 安装aws命令 https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
* 配置aws https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html  key找管理员要
* `An error occurred (AccessDeniedException) when calling the DescribeCluster operation: User:  is not authorized to perform: eks:DescribeCluster on resource:    :cluster/k8s-rdqa`. 在控制台给用户添加eks权限，都点上就行，不知道是哪个
* `An error occurred (AccessDeniedException) when calling the CreateCluster operation: User: arn:aws:iam:: is not authorized to perform: eks:CreateCluster on resource: ` 给用户user添加相应的策略（ eks:CreateCluster) iam:PassRole -- 单独加不管用，选了所有就好了。
* https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html 按照这个开就可以了
* node group: https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html#create-worker-node-role
* `NodeCreationFailure-> Unhealthy nodes in the kubernetes cluster`:  https://stackoverflow.com/questions/65807859/nodecreationfailure-unhealthy-nodes-in-the-kubernetes-cluster
* `NodeCreationFailure	Instances failed to join the kubernetes cluster`
* ec2实例如果没开ssh，后面就不能开了，所以最好建立的时候开一下
* ec2开了也登不上： 给自动生成的安全组加一个tcp全部可入，就可以ssh登录了， 自动生成的不好使
* ec2机器ping不通: 
* kubelet 的log在 /var/log/messages. `journalctl -u kubelet`
* eks 创建：`getting availability zones: getting availability zones for us-east-1: UnauthorizedOperation: You are not authorized to perform this operation.` : https://stackoverflow.com/questions/60438285/error-getting-availability-zones-when-trying-to-create-eks-cluster
* --ssh-public-key 这个指的是秘钥对的名字。。。。。


# aliyun

* docker login进不去，竟然是挂了vpn导致。。。。
* 如果不让直接 -p 输入密码 :  https://gitlab.com/gitlab-org/gitlab-runner/-/issues/2861

# kubectl get sc

强制删除
kubectl delete pods <pod> --grace-period=0 --force


 kubectl get pods | grep redash |  awk '{print $1}' | xargs kubectl delete pod --grace-period=0 --force


# helm

redis.gloabal.XXX 不好使， 要 : : : 
 
 
# 看node上都有神马pod
 
 `kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces`

