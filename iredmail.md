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

