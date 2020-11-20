# 指定hosts

ansible-playbook ./kafka.yml -i ./hosts/


# 使用账户

ansible-playbook ./kafka.yml -i ./hosts/ -b --become-user=work



## run tags

ansible-playbook ./kafka.yml -i ./hosts/ -b --become-user=work --tags deploy


## 不提示ssh的knowns hosts

根目录的ansible.cfg: 
```
[defaults]
pipelining = True
allow_world_readable_tmpfiles = True
host_key_checking = False
```


## 用root用户登录机器

--user=foo
