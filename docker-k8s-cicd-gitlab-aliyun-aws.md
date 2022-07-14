aws
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

### eksctl 

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

## 对等连接

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



aliyun
-------------

* docker login进不去，竟然是挂了vpn导致。。。。
* 如果不让直接 -p 输入密码 :  https://gitlab.com/gitlab-org/gitlab-runner/-/issues/2861.    
* 拉私有镜像： `kubectl create secret docker-registry regsecret --docker-server=registry.cn-zhangjiakou.aliyuncs.com   --docker-username=xxxx --docker-password=xxxxxxx` 注意image不要带https!
* 私网slb ：` service.beta.kubernetes.io/alibaba-cloud-loadbalancer-address-type: "intranet"`. https://help.aliyun.com/document_detail/86531.htm?spm=a2c4g.11186623.0.0.2bacc00dVOK3Y7#task-1425948
* 磁盘不够用：查是
 * /var/log/message太大，占了10g，7天2g，解决方法：修改/etc/logrotate.conf，rotate 4 -> 1 然后 `systemctl restart rsyslog` ---未验证
 * 镜像太多`/run/containerd/io.containerd.runtime.v2.task/k8s.io` `cat /etc/kubernetes/kubelet-customized-args.conf` https://segmentfault.com/a/1190000022163856
* 登录k8s虚机：1. 虚机开不了外网ip，需用eip，好的办法是从另一个虚机跳过去， eip和外网ip比，eip多点维护费
* k8s会用一个eip绑定到slb上，不知为何，收费！
* k8s开了一个eip绑定到nat，不知道干什么用的 ??
* rabbitmq & promtheus: https://help.aliyun.com/document_detail/161843.html?spm=a311a.7996332.0.0.2bc93080wZ9hk2
* 拉不了私有镜像： 没有aliyun-acr-credential-helper， 安装一下就好了，猜测：老的集群建立时，点了ingress,我猜这个会自动安装这个app
* 建不了抢占式实例：账号没权限，建了包月突发，贵一点
* 跳板机创建：需要新建一个交换机，注意机器和交换机是绑定的，一个vpc的交换机是互通的
* ssh到k8s机器： ssh到跳板机，copy pem到跳板机，在ssh -i pem到k8s node。
* 跨区拉镜像：
  * 安装aliyun-acr-credential-helper
  * 改acr-configuration
  * 加regionID
```
- instanceId: ""
  regionId: "cn-zhangjiakou"
```
* oss挂载不了
  * 增加机器到两台，解决CSI插件起不起来的问题，即oss挂载不了
  * 给k8s资源组授权oss全访问权限
* mysql访问不了
  * 白名单加pod白名单 172.19.224.0/20
  * 建数据库后，要给账号授权
* rabbit报vhost没权限： RAM加amqp权限
* 

## pvc

* https://help.aliyun.com/document_detail/134722.html
* pv的 aksk是 oss的
* 按照官网方法2， 要加label，文档竟然是错误的，没加。。。。




## k8s to sls

* https://help.aliyun.com/document_detail/87540.html?spm=5176.smartservice_service_robot-chat.help.dexternal.6c674b0dRyuKQp 
* project不要新建，用建立k8s自带的，要不然找不到机器组
* 示例配置(废弃，新的直接搞yaml就可以，yaml的logstore会自动建立，更少的控制台操作）

```json
{
    "inputs": [
        {
            "detail": {
                "Stderr": true,
                "IncludeLabel": {
                    "io.kubernetes.pod.namespace": "YOUR NS"
                },
                "Stdout": true
            },
            "type": "service_docker_stdout"
        }
    ]
}
```
- 告警https://help.aliyun.com/document_detail/207609.html， 一个文件默认只被一个logstore，要想支持一个文件入多个logstore，需要改配置
- logstore删了，再重启dep，有时不能重建，解决方案： 重启logtail-ds的pod.
- 告警邮件展示自动标注： 发送内容里加一个annotations: ${annotations} <br>
- 监控pod启动： k8s-event库，添加`* and eventId.reason: Started`相关告警即可， 用自带的那个好像不好使（也许好使），但是即使好使，也无法展现podname

## sls 数据加工

* 数据加工建立后，报错，python脚本说没有你要的logstore： 自己建一个。。。。。
* json展开： https://help.aliyun.com/document_detail/125488.html#section-o7x-7rl-2qh
* srs的log带了颜色，数据加工不能简单排除，最后用文件解决这个问题，但srs不会自动建立目录，需要注意


gitlab
-------------------------

* runner 显示$是正常的，怕泄漏
* docker 里build不了docker: https://gitlab.com/gitlab-org/gitlab-runner/-/issues/1986
* docker login 不行，要stdin: https://gitlab.com/gitlab-org/gitlab-runner/-/issues/2861
* docker build 不成功： srs 要 `-f ./trunk/Dockerfile .`
* docker push 不上去：runner挂了代理。。。。
* srs : `sudo docker build-t rdqa/zzzzzz:test  -f ./trunk/Dockerfile .`
* `docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]` 
* `docker push registry.cn-zhangjiakou.aliyuncs.com/xxxx/zzzzz:test`
* 引用 issue https://docs.gitlab.com/ee/user/project/issues/crosslinking_issues.html
* gitlabci使用私有仓库做基础镜像：https://docs.gitlab.com/ee/ci/docker/using_docker_images.html#determine-your-docker_auth_config-data
* `standard_init_linux.go:211: exec user process caused "exec format error" ` 在m1编的镜像 linux用不了 : https://gitlab.com/gitlab-org/omnibus-gitlab/-/issues/4558
* 在job前加.可以跳过阶段
* gitlab改基础镜像的entry：`image:entrypoint`
* gitlab覆盖率需要在cicd配置`total:\s+\(statements\)\s+(\d+.\d+\%)`
* CI_COMMIT_REF_NAME : 新代码的分支名
* `To push a commit without triggering a pipeline, add [ci skip] or [skip ci], using any capitalization, to your commit message.`
* gitlab runner取镜像，sha变了， latest不会同步，解决：用tag。。。
* centos瘦身： `yum install a && yum install b 改为 install a b  &&  yum clean all && rm -rf /var/cache/yum`
* 不管downstream 的状态直接success： 得加`strategy: depend`
* docker prune 只清没tag的， 有tag得 docker image rm
* `No permissions to trigger downstream pipeline` 提升commiter的权限就好了，不知道为啥
* debug ci job: 
```
job_name:
  variables:
    CI_DEBUG_TRACE: "true"
```
* color :  `[-red-]`  `[-green-]` [-red-] [-green-]
* 当 only:pushes， 当打tag，也会触发pushes， 如果打tag不触发pushes，要 except tag   ：   only:pushes /n except: tags     pushes = push + tags

docker 
-------------------

* docker login不要sudo .... 
* 没办法给一个启动的docker expose端口 : https://stackoverflow.com/questions/19897743/exposing-a-port-on-a-live-docker-container.  `-p 8935:1935`
* `docker run -v /xxxxxxxxx/trunk:/srstrunk --name srs3  -P   -dit  srsdev /bin/bash` 
* 起个centos：`docker run -it centos`
* 有些镜像设置了启动命令不是shell，你要进sh，怎么办？`docker run --entrypoint '/bin/sh' -it  k8s.gcr.io/kustomize/kustomize:v3.8.7`
* docker pull超出dockerhub次数： https://cloud.tencent.com/developer/article/1701933（未验证)
* alpine/git没有bash， docker:latest没有git， bash+git就乖乖centos+git把
* 地方不够，dockerbuild: 重启不了docker的话，删了重装。。。。
```
 :~/golang-image-extra$ cat /etc/docker/daemon.json
{
                    "data-root": "/data",
                                "storage-driver": "overlay2"
}
```  

* overlay目录清理:`sudo docker system prune -a -f`


## docker开发srs

* entrypoint 和cmd 如果都有，那cmd就是entrypoint的参数。。。 https://www.cnblogs.com/sparkdev/p/8461576.html
* 没有cmd的镜像也可以跑起来，注意mount的话， /xxxx:/xxxx  都要是绝对目录，否则不行！ https://stackoverflow.com/questions/18878216/docker-how-to-live-sync-host-folder-with-container-folder
* dockerfile 的expose 是给 docker直接打开的端口， 宿主机无法访问，所以得 -P 或 -p 

 
 
 
cicd
---------------

 * argo :https://zhuanlan.zhihu.com/p/181692322
 * https://github.com/spinnaker/spinnaker
 * weave
 * tekton
 * jenkins X
 
 ## argocd
 
* 先在gitlab建立自己的token， 然后argo 连接的时候， username:就是gitlab你的username， token就是刚才的token，skip version ssl
* argo的git url是取里面的yaml进行部署，不是sourcecode，如果要递归，需要点一下recursive 
* sync不成功的话，ns没建，他不会自己建ns
* https://medium.com/@andrew.kaczynski/gitops-in-kubernetes-argo-cd-and-gitlab-ci-cd-5828c8eb34d6  比较好
* 应该会自动忽略.的部署： https://github.com/argoproj/argo-cd/issues/2638
* argocd ns删除不掉： ` kubectl get Application -n argocd` 然后delete，但是不行，改一下finilizer: `finalizer: [] `. done.
* argocd用helm 看不到，是因为：argo执行 `helm template . <options> | kubectl apply -f - `
* 覆盖values.yaml : https://argo-cd.readthedocs.io/en/stable/user-guide/helm/#helm-parameters (未成功）


## argo rollout

### aliyun asm

* 我的ack版本不支持asm，等了一周后支持了，然后开始调试
* rollout一点反应也没有，发现是argo rollout没安装。。。。。
* 安装后提示 vs找不到，这是因为asm 数据平面要找控制平面，需要打通：https://help.aliyun.com/document_detail/336919.htm?spm=5176.13895322.help.dexternal.43de5fcfhw5OAi
* 按照文档安装后，提示ram没权限，我给了个AliyunASMFullAccess 好使了，asm权限也改成了管理员


k8s 
------------- 
  
* kecm换editor： `KUBE_EDITOR="nano"`
* 看node上都有神马pod : `kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces`
* 获取headless ip :  `dig srv srs-origin-service.wzjinlei.svc.cluster.local`    SRV.NS.svc.cluster.local.
* 强制删除: `kubectl delete pods <pod> --grace-period=0 --force` `kubectl get pods | grep redash |  awk '{print $1}' | xargs kubectl delete pod --grace-period=0 --force`
* kubectl get sc
* pod 给pod 发信号，用于logrotate: https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/#configure-a-pod
* 看 request: https://github.com/kubernetes/kubernetes/issues/17512
*  `kubectl get po --all-namespaces -o=jsonpath="{range .items[*]}{.metadata.namespace}:{.metadata.name}{''}{range .spec.containers[*]}  {.name}:{.resources.requests.cpu}{'\n'}{end}{'\n'}{end}"  | grep -e ':\d*m'`
*  `alias util='kubectl get nodes --no-headers | awk '\''{print $1}'\'' | xargs -I {} sh -c '\''echo {} ; kubectl describe node {} | grep Allocated -A 5 | grep -ve Event -ve Allocated -ve percent -ve -- ; echo '\'''`.  `kubectl describe nodes `
* 给service改为loadbalancer（from argo): ` kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'`
* k8s配置尴尬局面： https://blog.argoproj.io/the-state-of-kubernetes-configuration-management-d8b06c1205  
* `访问模式有：
ReadWriteOnce
卷可以被一个节点以读写方式挂载。 ReadWriteOnce 访问模式也允许运行在同一节点上的多个 Pod 访问卷。
ReadOnlyMany
卷可以被多个节点以只读方式挂载。
ReadWriteMany
卷可以被多个节点以读写方式挂载。
ReadWriteOncePod
卷可以被单个 Pod 以读写方式挂载。 如果你想确保整个集群中只有一个 Pod 可以读取或写入该 PVC， 请使用ReadWriteOncePod 访问模式。这只支持 CSI 卷以及需要 Kubernetes 1.22 以上版本。`
* 获取客户端ip： https://www.cnblogs.com/zisefeizhu/p/13262239.html. （未实验)
* -o yaml customize: `https://www.qikqiak.com/post/boosting-kubeclt-productivity/`
* 定制指标扩缩容： https://www.qikqiak.com/post/build-k8s-app-with-custom-metrics/
* cp文件，前面不能用绝对路径 `k cp ns/pod:x.log    ./a`
* cp bin进行调试： https://blog.csdn.net/StephenLu0422/article/details/78900420  patch有问题，需要自己改一下，方法是可行的
* 笔记本打开一个pod的端口

```
# Change mongo-75f59d57f4-4nd6q to the name of the Pod
kubectl port-forward mongo-75f59d57f4-4nd6q 28015:27017
```


kustomize
---------

* docker的kustomize镜像基于alpine，起了他的bin，所以不能做基础镜像，用centos7 + curl
* 使用简介：https://blog.stack-labs.com/code/kustomize-101/


helm
---------

* debug: `helm template --debug`比 helm lint方便， lint解决不了，用template看看原因
* template用于configmap， include用于其他，好像include是用yaml的，configmap 只能用template，具体细节没研究明白
* 遍历0123：`{{range $i, $e := until (.Values.replicas | int)}}  {{$i}} {{end}}` 如果报float的话，用int
* redis-helm: redis.gloabal.XXX 不好使， 要 : : : 
* upgrade无法升级pvc `: spec.persistentvolumesource is immutable after creation` : 要把所有的用pvc的pod停了就行 scale.
* `{{ .Release.Namespace }}`
* tips: https://www.qikqiak.com/post/helm-chart-tips-and-tricks/
* 不删pvc： https://helm.sh/docs/howto/charts_tips_and_tricks/  (未验证)
* comment:
```
{{- /*
This is a comment.
*/}}
type: frobnitz
```
* template 好像不能渲染里面的if，巨坑，折腾一下午，得用include xxx .
* helm 因为 超时导致failed，然后upgrade不成功， helm history 可以看原因，解决：https://jacky-jiang.medium.com/how-to-fix-helm-upgrade-error-has-no-deployed-releases-mystery-3dd67b2eb126


prometheus
------

* https://www.qikqiak.com/k8s-book/docs/58.Prometheus%20Operator.html
* https://juejin.cn/post/6844903908251451406

其他
----------
 

## mac安装kubectl

https://kubernetes.io/zh/docs/tasks/tools/install-kubectl-macos/

创建 ~/.kube/config 文件，将控制台的kubeconfig拷入即可

 
```
[jinleileiking:~/.kube] master(+27/-27)* ± kubectl cluster-info
Kubernetes control plane is running at xxxxxx
metrics-server is running at xxxxx/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at xxxxxxxx/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```


## 效率提升

如果你用zsh， oh-my-zsh，

使用ohmyzsh的plugin：

我用zplug管理：改~/.vimrc 装zplug，并在vimrc添加：

`zplug "plugins/kubectl",   from:oh-my-zsh`

然后：

```
[jinleileiking:~/.kube] master(+27/-27)* ± alias | grep kube | head
k=kubectl
kaf='kubectl apply -f'
kca='_kca(){ kubectl "$@" --all-namespaces;  unset -f _kca; }; _kca'
kccc='kubectl config current-context'
kcdc='kubectl config delete-context'
kcgc='kubectl config get-contexts'
kcn='kubectl config set-context --current --namespace'
kcp='kubectl cp'
kcsc='kubectl config set-context'
kcuc='kubectl config use-context'
```

这样就方便多了

kubens  用于换环境
kubectx 用于换namespace
