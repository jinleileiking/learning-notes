# 指定hosts

ansible-playbook ./kafka.yml -i ./hosts/


# 使用账户

ansible-playbook ./kafka.yml -i ./hosts/ -b --become-user=work
