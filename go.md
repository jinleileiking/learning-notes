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
