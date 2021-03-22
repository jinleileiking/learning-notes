kubectl get sc

强制删除
kubectl delete pods <pod> --grace-period=0 --force


 kubectl get pods | grep redash |  awk '{print $1}' | xargs kubectl delete pod --grace-period=0 --force


# helm

redis.gloabal.XXX 不好使， 要 : : : 
