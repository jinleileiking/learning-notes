# 取得sshkey

```
terraform show -json | \
jq -r '.values.root_module.resources[].values | select(.private_key_pem) |.private_key_pem'
```


# aliyun dns 无权限

需要在用户那加上dns的权限


# cloudfront 报alias错

https://aws.amazon.com/cn/premiumsupport/knowledge-center/resolve-cnamealreadyexists-error/

就是，aws会验证alias的dns，需要从阿里把这个dns删掉



# cloudwatch alarm

- 默认值在log_metric_filter里，metric_transformation_default_value，而不在metric_alarm里

# eks添加子网

开发中，遇到1a没有机器了，所以要多加一个子网

- 直接改vpc的话，会不成功，说eks要重新建立，但重建说eks重复
- 执行apply后，会发现vpn改好了，eks状态不对了
- 手动在控制台上加了设备组的子网
- `terraform state rm  module.eks.aws_eks_cluster.this`
- 修改k8s provider为file ` config_path    = "~/.kube/config"`
- ` terraform import     -var="region=ap-south-1" -var="env=prod" "module.eks.aws_eks_cluster.this[0]" eks-ap-south-1` 这里注意，要有`[0]`，命令行会有 aws_eks_cluster的提示，且在statetf里能找到对应为成功（这两部其实可以省略）
- 修改tf文件里的subnetid为两个，不要用vpc里面的三个
- rm state会删除oidc，但执行apply的话会补回来

# 忽略某些tf文件

结论：目前不可行，workaround：

1. https://github.com/hashicorp/terraform/issues/27360 lifecycle目前不能给module进行ignore
2. -exclude目前也没实现只能搞一堆-target
