# aws 开k8s

* role要创建k8s-cluster-policy, k8s-policy不管用
* 子网要开两个可用区
* https://aws.amazon.com/cn/blogs/china/approached-fargate-hands-on-configuration-belongs-to-their-own-fargate-cluster/ 




# kubectl get sc

强制删除
kubectl delete pods <pod> --grace-period=0 --force


 kubectl get pods | grep redash |  awk '{print $1}' | xargs kubectl delete pod --grace-period=0 --force


# helm

redis.gloabal.XXX 不好使， 要 : : : 
 
 
# 看node上都有神马pod
 
 `kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces`

