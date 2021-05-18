# 部署

注意先改dockerize的conf文件， 如果重启，注意考虑删除data目录


```
HOSTNAME=mail.nicegoodthings.com
FIRST_MAIL_DOMAIN=nicegoodthings.com
```


```
docker run \
    --rm \
    --name iredmail \
    --env-file iredmail-docker.conf \
    --hostname mail.nicegoodthings.com \
    -p 8180:80 \
    -p 8443:443 \
    -p 110:110 \
    -p 995:995 \
    -p 143:143 \
    -p 993:993 \
    -p 25:25 \
    -p 465:465 \
    -p 587:587 \
    -v /iredmail/data/backup:/var/vmail/backup \
    -v /iredmail/data/mailboxes:/var/vmail/vmail1 \
    -v /iredmail/data/mlmmj:/var/vmail/mlmmj \
    -v /iredmail/data/mlmmj-archive:/var/vmail/mlmmj-archive \
    -v /iredmail/data/imapsieve_copy:/var/vmail/imapsieve_copy \
    -v /iredmail/data/custom:/opt/iredmail/custom \
    -v /iredmail/data/ssl:/opt/iredmail/ssl \
    -v /iredmail/data/mysql:/var/lib/mysql \
    -v /iredmail/data/clamav:/var/lib/clamav \
    -v /iredmail/data/sa_rules:/var/lib/spamassassin \
    -v /iredmail/data/postfix_queue:/var/spool/postfix \
    iredmail/mariadb:stable
```


# 改nginx

docker有问题, 443 如果定制为8443  301 redirect不带端口，会有问题，直接开8443 http得了

```
#listen 443 ssl http2;
    listen 443;
```

# 添加用户

进入iredadmin加用户就行了，sql好像不好使

# mx 记录

```
[backend@yanggc ~]$ dig mx nicegoodthings.com

nicegoodthings.com.     126     IN      MX      10 mail.nicegoodthings.com.
```

```
mail.nicegoodthings.com. 600    IN      A       your ip
```


阿里云： 

```
主机记录： mail    记录类型 A    记录值： yourip
            @            MX    记录值  mail.yourdomain.com
```

即   yourdoamin.com  MX -->  mail.yourdomain.com  A --> your serverip





# postfix debug

# php debug

