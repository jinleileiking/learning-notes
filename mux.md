# publish

 rtmp://global-live.mux.com:5222/app/a2541cc8-66a8-82a5-0654-14448a5d24c7

global-live.mux.com

https://dnschecker.org/#A/global-live.mux.com

```
# dig global-live.mux.com +short
geo-live.mux.com.
global-live-legacy.mux.com.
35.186.203.115
```

```
Mountain View CA, United States
Google 	35.237.91.193
34.75.172.244
34.74.234.170	
Berkeley, US
Quad9 	34.75.172.244
35.237.91.193
34.74.234.170	
Holtsville NY, United States
OpenDNS 	35.237.91.193
34.74.234.170
34.75.172.244	
Miami, United States
AT&T Services 	35.237.91.193
34.74.234.170
34.75.172.244	
Brooklyn, United States
Verizon Fios Business 	35.237.91.193
34.75.172.244
34.74.234.170	
Canoga Park, CA, United States
Sprint 	34.74.234.170
35.237.91.193
34.75.172.244	
San Jose, United States
Corporate West Computer Systems 	35.185.255.240
35.199.149.94	
Toronto, Canada
Cogecodata 	35.237.91.193
34.74.234.170
34.75.172.244
```

各个洲返回的ip不一样，用的gcp


推流地址：  rtmp://global-live.mux.com:5222/app

看到用的不是标准1935，而是5222， 猜测和lb有关，

https://cloud.google.com/load-balancing/docs/tcp?hl=zh_cn， 果然里面接口有5222

XMPP/Jabber - client connection	 是 5222



San Jose, United States


# play 

ffplay  https://stream.mux.com/qulZyQvZzhQbR5YsZUiOL8WJWnozSFebiJBwmlCQfXU.m3u8


推拉流域名不一样， 流名也不一样
