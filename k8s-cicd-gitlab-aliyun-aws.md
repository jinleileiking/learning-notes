# aws 

## 开k8s

### 控制台

* role要创建k8s-cluster-policy, k8s-policy不管用
* 子网要开两个可用区

### aws命令

* 不要在控制台点创建，要用aws这个命令创建 --- 事实证明也不好使
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
* ??????  ec2机器ping不通: 
* kubelet 的log在 /var/log/messages. `journalctl -u kubelet`
* ??????    控制台看不到node`您的当前用户或角色无权访问此 EKS 集群上的 Kubernetes 对象`


### eksctl 

* eks 创建：`getting availability zones: getting availability zones for us-east-1: UnauthorizedOperation: You are not authorized to perform this operation.` : https://stackoverflow.com/questions/60438285/error-getting-availability-zones-when-trying-to-create-eks-cluster
  * --ssh-public-key 这个指的是秘钥对的名字。。。。。
* eks 开直接就成功了，但机器是两台而且是m5 large... 使用小机器： https://eksctl.io/usage/creating-and-managing-clusters/ 尝试用t2不行，改成t3a.micro
  * t3a pod找不到node创建：初始配置一个node只能跑4个pod。。。。。 https://github.com/awslabs/amazon-eks-ami/blob/master/files/eni-max-pods.txt
  * t3a.small 8 , t3.small 11

## k8s使用

* 暴露的service访问不了
  * 登录虚机可以curl
  * 过了一会就能进了，估计是service生效有时间？

## cloudwatch

* `CreateLogGroup API responded with error='AccessDeniedException'` 
 
 


# gitlab

* runner 显示$是正常的，怕泄漏
* docker 里build不了docker: https://gitlab.com/gitlab-org/gitlab-runner/-/issues/1986
* docker login 不行，要stdin: https://gitlab.com/gitlab-org/gitlab-runner/-/issues/2861
* docker build 不成功： srs 要 `-f ./trunk/Dockerfile .`
* docker push 不上去：runner挂了代理。。。。
* srs : `sudo docker build-t rdqa/zzzzzz:test  -f ./trunk/Dockerfile .`
* `docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]` 
* `docker push registry.cn-zhangjiakou.aliyuncs.com/xxxx/zzzzz:test`
* 引用 issue https://docs.gitlab.com/ee/user/project/issues/crosslinking_issues.html



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

