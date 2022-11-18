# 取得sshkey

```
terraform show -json | \
jq -r '.values.root_module.resources[].values | select(.private_key_pem) |.private_key_pem'
```


# aliyun dns 无权限

需要在用户那加上dns的权限
