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

* 安装这个安装即可:https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs-FluentBit.html
* `CreateLogGroup API responded with error='AccessDeniedException'` : create 文件，加入 cloudwatch配置，就好了，期间还加了一次cloudwatchlog all的权限
* 改fluent-bit-config应该就可以收集其他日志了
 
 
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

## pvc

* https://help.aliyun.com/document_detail/134722.html
* pv的 aksk是 oss的
* 按照官网方法2， 要加label，文档竟然是错误的，没加。。。。


## k8s to sls

* https://help.aliyun.com/document_detail/66658.html
* project不要新建，用建立k8s自带的，要不然找不到机器组
* 示例配置

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


docker 
-------------------

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
 
 ## argo
 
* 先在gitlab建立自己的token， 然后argo 连接的时候， username:就是gitlab你的username， token就是刚才的token，skip version ssl
* argo的git url是取里面的yaml进行部署，不是sourcecode，如果要递归，需要点一下recursive 
* sync不成功的话，ns没建，他不会自己建ns
* https://medium.com/@andrew.kaczynski/gitops-in-kubernetes-argo-cd-and-gitlab-ci-cd-5828c8eb34d6  比较好
* 应该会自动忽略.的部署： https://github.com/argoproj/argo-cd/issues/2638
* argocd ns删除不掉： ` kubectl get Application -n argocd` 然后delete，但是不行，改一下finilizer: `finalizer: [] `. done.
* argocd用helm 看不到，是因为：argo执行 `helm template . <options> | kubectl apply -f - `

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


prometheus
------

* https://www.qikqiak.com/k8s-book/docs/58.Prometheus%20Operator.html

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
