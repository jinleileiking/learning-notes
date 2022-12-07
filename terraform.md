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
