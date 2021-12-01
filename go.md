# go 学习笔记

第六版 下 预览

span里面有obj，一共67种class， class 0 就是一个大内存

大块内存=span 


# http client

```
transport.go    (t *Transport) dialCon

readloop:

 _, err := pc.br.Peek(1) --这里只看一下，没数据也没事，eof也没事
 
rc := <-pc.reqch      reqch     chan requestAndChan // written by roundTrip; read by readLoop  

pc.readResponse

```

所以就是read先等write信号，write完，跑read

对于eof pc.readResponse`挂了

```
   2172 ▸   ▸   resp, err = ReadResponse(pc.br, rc.req)¬
|   2173 ▸   ▸   if err != nil {¬
|   2174 ▸   ▸   ▸   return¬
|   2175 ▸   ▸   }¬
```

应该是这里返回eof了


# test & gin

* go-sqlmock + gin : https://blog.csdn.net/vvstormhao/article/details/112376130
 * SkipInitializeWithVersion 需要设置为true
 * 	pool.SetMaxIdleConns(3) pool.SetMaxOpenConns(3) 这个不能设置否则跑不通
* gin 里面传递数据库信息： https://github.com/gin-gonic/gin/issues/932#issuecomment-306242400
* gin 优雅关闭 https://github.com/valord577/webdav/blob/main/cmd/serv.go#L69
* 用group的话， 要把最后留出来，group里类似 GET("/")就不好使了
* setup, teardown: https://geektutu.com/post/quick-go-test.html
* time.Now() or time.Now().UTC() 没必要调用utc https://stackoverflow.com/questions/44873825/how-to-get-timestamp-of-utc-time-with-golang
* 跳过case：  t.Skip("Skipping testing in CI environment")
* 判断是否在go test中：https://stackoverflow.com/questions/14249217/how-do-i-know-im-running-within-go-test
