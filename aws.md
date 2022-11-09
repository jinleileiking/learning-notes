eks
----------------

* 加磁盘:`https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/add-instance-store-volumes.html`
* 开k8s
  * 控制台
    * role要创建k8s-cluster-policy, k8s-policy不管用
    * 子网要开两个可用区
  * aws命令
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
* 访问阿里云镜像: `kubectl create secret docker-registry <名称> \
  --docker-server=DOCKER_REGISTRY_SERVER \
  --docker-username=DOCKER_USER \
  --docker-password=DOCKER_PASSWORD \
  --docker-email=DOCKER_EMAIL`
  * 注意密码看看对不对
  * 通过` journalctl -u kubelet --no-pager -f` 来看log
  * 在deployment加 
  ```
  imagePullSecrets:
    - name: myregistrykey
  ```
  * https://kubernetes.io/zh/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
* 挂载云盘
  - aws默认支持云盘，是最方便的方式了，云盘默认有gp2的动态存储卷，不用声明pvc，pv，直接把deployment改为statefulsets，加个templatepvc，每一个pod就有一个云盘了，对于coredump，好使。
* grafana 采集 prometheus metics
  - 这套系统不用部署prometheus， aws可以直接把metrics采集到cloudwatch
  - https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup-configure.html
  - https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config
  - 记得加 metric_path: /metrics，默认好使是/
  - kaf yaml后，要del pod才生效，光升配置没用，好像

## 给node保留cpu内存

https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/#kube-reserved
https://eksctl.io/usage/customizing-the-kubelet/

注意只能是不managed了。有方法github弄managed，比较麻烦

##  eksctl 

* eks 创建：`getting availability zones: getting availability zones for us-east-1: UnauthorizedOperation: You are not authorized to perform this operation.` : https://stackoverflow.com/questions/60438285/error-getting-availability-zones-when-trying-to-create-eks-cluster
  * --ssh-public-key 这个指的是秘钥对的名字。。。。。
* eks 开直接就成功了，但机器是两台而且是m5 large... 使用小机器： https://eksctl.io/usage/creating-and-managing-clusters/ 尝试用t2不行，改成t3a.micro
  * t3a pod找不到node创建：初始配置一个node只能跑4个pod。。。。。 https://github.com/awslabs/amazon-eks-ami/blob/master/files/eni-max-pods.txt
  * t3a.small 8 , t3.small 11
* 没有机器
  * 如果要有ssh，那你的子网得有外网访问权限
  * eksctl会从你配置的子网zone获取机器，也不会遍历，如果选中了1e，1e没有，那就失败了，所以要从ecs界面，看看哪两个zone有机器，然后创建public子网（需要两个），这样才能继续
* 在界面看不到机器资源
  * aws configure设置eksctl的账号，需要用子账号登进去，才能看到相关
* 没机器：`reached your quata for maximum Fleet Requests for this account.` 

## k8s使用

* 暴露的service访问不了
  * 登录虚机可以curl
  * 过了一会就能进了，估计是service生效有时间？
* statefulsset 的pvc是有磁盘属性的，也就是说一个pod，如果在A机器上启动，那就一直在A了，无法调配，需要开启feature gate，才能自动删除ebs

## nlb

* nlb -> targetgroupbinding -> nodeport
* 遇到health check不通，看一看nodegroup的安全组，安全组端口要通，加192.168 172.x.x.x

## cloudwatch

* 安装这个安装即可:https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs-FluentBit.html
* https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-EKS-quickstart.html
* https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/Container-Insights-view-metrics.html
* https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html
* `CreateLogGroup API responded with error='AccessDeniedException'` : create 文件，加入 cloudwatch配置，就好了，期间还加了一次cloudwatchlog all的权限
```
将所需的策略附加到 Worker 节点的 IAM 角色

通过以下网址打开 Amazon EC2 控制台：https://console.aws.amazon.com/ec2/。

选择其中的一个 Worker 节点实例，然后在描述中选择 IAM 角色。

在 IAM 角色页面上，选择 Attach policies（附加策略）。

在策略列表中，选中 CloudWatchAgentServerPolicy 旁边的复选框。如有必要，请使用搜索框查找该策略。

选择 Attach policies（附上策略）。
```
- https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/Container-Insights-prerequisites.html
- aws docker 默认rotate： https://github.com/awslabs/amazon-eks-ami/blob/master/files/docker-daemon.json#L4-L6 

* grafana不支持变量：https://github.com/grafana/grafana/issues/24603  用正则。。。。
* recap
  * https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-EKS-quickstart.html 这个收集applog到application，和host，dataplane
  * https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html 这个收集perfermance 如何变为metrics的呢？ 

##  对等连接

- https://docs.aws.amazon.com/zh_cn/vpc/latest/peering/vpc-peering-routing.html 按照这个，把两个vpc打通，注意要配置两个路由表
- 还要给k8s的node加上icmp安全组，才可以ping通


## 挂载efs

- created IAM Open ID Connect provider： ` eksctl utils associate-iam-oidc-provider --cluster prod-k8s --approve`
- policy: 
```
aws iam create-policy \
    --policy-name AmazonEKS_EFS_CSI_Driver_Policy \
    --policy-document file://iam-policy-example.json`
```
- iam sa
```
eksctl create iamserviceaccount \
    --name efs-csi-controller-sa \
    --namespace kube-system \
    --cluster prod-k8s \
    --attach-policy-arn arn:aws:iam::802625923695:policy/AmazonEKS_EFS_CSI_Driver_Policy \
    --approve \
    --override-existing-serviceaccounts \
    --region us-east-1
```
- helm install
```
helm upgrade -i aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver \
    --namespace kube-system \
    --set image.repository=602401143452.dkr.ecr.us-east-1.amazonaws.com/eks/aws-efs-csi-driver \
    --set controller.serviceAccount.create=false \
    --set controller.serviceAccount.name=efs-csi-controller-sa
```
- https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html

## fsx

- 注意，resouce：s3 改为 * 就行了！！！！！

## ebs

- 一个机器有最大磁盘数目挂载量，20个左右https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/volume_limits.html#linux-specific-volume-limits

## prometheus

- https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-ingest-metrics-new-Prometheus.html 
- https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-query-standalone-grafana.html
- https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/
- https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus#scraping-pod-metrics-via-annotations
- https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config
- https://stackoverflow.com/questions/59866342/prometheus-dynamic-metrics-path
- helm status prometheus, 然后port-forward
- 测试环境接入跨区prom，需要确保oidc有police策略，不用官网那个脚本，直接在信任里添加新的eks信息就行，注意里面有ns信息
<img width="446" alt="image" src="https://user-images.githubusercontent.com/93968/199686019-33f52cf1-3a3e-406c-887f-5f9210d6505b.png">


## ca

- 曾经在控制台把最大机器改成了1，扩容不行，改成10，ca也不行，重启pod好了。


## 一次pod terminating的问题

https://github.com/kubernetes/autoscaler/issues/4966, 排查，kubelet log：
```
Jun 15 06:55:57 ip-192-168-44-229.ec2.internal kubelet[4195]: I0615 06:55:57.895419    4195 reconciler.go:196] "operationExecutor.UnmountVolume started for volume \"volume-hls-disk\" (UniqueName: \"kubernetes.io/csi/fsx.csi.aws.com^fs-000f8f664d17faf82\") pod \"b2a61441-8129-4372-946c-cea170ae9979\" (UID: \"b2a61441-8129-4372-946c-cea170ae9979\") "
Jun 15 06:55:57 ip-192-168-44-229.ec2.internal kubelet[4195]: E0615 06:55:57.895504    4195 nestedpendingoperations.go:301] Operation for "{volumeName:kubernetes.io/csi/fsx.csi.aws.com^fs-000f8f664d17faf82 podName:b2a61441-8129-4372-946c-cea170ae9979 nodeName:}" failed. No retries permitted until 2022-06-15 06:57:59.895477052 +0000 UTC m=+84822.216273975 (durationBeforeRetry 2m2s). Error: "UnmountVolume.TearDown failed for volume \"volume-hls-disk\" (UniqueName: \"kubernetes.io/csi/fsx.csi.aws.com^fs-000f8f664d17faf82\") pod \"b2a61441-8129-4372-946c-cea170ae9979\" (UID: \"b2a61441-8129-4372-946c-cea170ae9979\") : kubernetes.io/csi: mounter.SetUpAt failed to get CSI client: driver name fsx.csi.aws.com not found in the list of registered CSI drivers"
Jun 15 06:56:16 ip-192-168-44-229.ec2.internal kubelet[4195]: WARNING: 2022/06/15 06:56:16 grpc: addrConn.createTransport failed to connect to {/var/lib/kubelet/plugins/fsx.csi.aws.com/csi.sock  <nil> 0 <nil>}. Err :connection error: desc = "transport: Error while dialing dial unix /var/lib/kubelet/plugins/fsx.csi.aws.com/csi.sock: connect: connection refused". Reconnecting...
```

果然是，pod要detach pvc不行，为啥呢，因为之前把fsx的按照搞过一次，有些估计不行了


## argo, crossplane alpha1

```
❯ kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-aws:v1.10.1

kubectl crossplane: error: failed to create kube client: exec plugin: invalid apiVersion "client.authentication.k8s.io/v1alpha1"
```

1. 升级aws cli
2. aws eks update-kubeconfig  --region=us-east-2 --name=rdqa-k8s


#  kinesis firehose

kinesis收不到fluent bit的数据：

1. 要用stern看所有pod的，当初只看了一个pod，导致错了。
2. 给nodegroup加kinesis firehose的权限，注意，不能是kinesis，必须是firehose，不能是eks的权限，必须是nodegroup的


#  kinisis datastreams

1. fluentbit 官网的文档是错的，应该用plugin：kinesis
2. lamda mac 用 pip3 install，必须打包，因为依赖request
3. rewrite filter reg必须是字符串，int不行！
4. 使用stdout output进行debug
5. https://aws.amazon.com/cn/blogs/china/build-a-logging-system-with-fluent-bit-and-amazon-opensearch-service/
6. 需更改region


## aos

1. 碰到使用t3 small，dashboard打不开，重新用t3 medium就行了。 

#  e2c

* 带宽 

```
The number reported is the number of bytes sent during the period. If you are using basic (5-minute) monitoring and the statistic is Sum, you can divide this number by 300 to find Bytes/second. If you have detailed (1-minute) monitoring and the statistic is Sum, divide it by 60.
```

即：sum值 / 300 * 8


 
