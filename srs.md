[[_TOC_]]

# 测试

srs单测基于c的测试，代码在src/utest

集成测试基于go test，代码在`3rdparty/srs-bench/*_test.go`


# 代码分析



代码基于：23730fa48387fc41d847e2ab69d65ee721676b8e  4.0release

## 启动：

```
SrsServer::listen()
  SrsBufferListener::listen
  

```


## 推流：

```
SrsBufferListener::on_tcp_client[srs_app_server.cpp]
  server->accept_client[srs_app_server.cpp]
     SrsRtmpConn::start()[srs_app_rtmp_conn.cpp]
     
SrsRtmpConn::cycle()
  SrsRtmpConn::do_cycle()
    rtmp->handshake()
    
    
    
      SrsComplexHandshake::handshake_with_client
    rtmp->connect_app(req) = SrsSimpleRtmpClient::connect_app()
    SrsRtmpConn::service_cycle()[srs_app_rtmp_conn.cpp]
      SrsRtmpServer::set_window_ack_size
        SrsProtocol::send_and_free_packet
          SrsProtocol::do_send_and_free_packet
            SrsProtocol::send_and_free_message
              SrsProtocol::send_and_free_messages
      while(true){
      SrsRtmpConn::stream_service_cycle
        _srs_sources->fetch_or_create(req, server, &source) --> source里存放着流的信息，作为重复推流，拉流对应关系的依据
        start_fmle_publish -> 这里会响应fconpublish给ffmpeg
        SrsRtmpConn::publishing
          SrsRtmpConn::acquire_publish
            SrsLiveSource::on_publish[srs_app_source.cpp]
              SrsOriginHub::on_publish
                SrsHls::on_publish[srs_app_hls.cpp] 
                  SrsHlsController::on_publish
                    SrsHlsMuxer::on_publish
          SrsRtmpConn::do_publishing
            SrsRtmpConn::handle_publish_message
              SrsRtmpConn::process_publish_message
                SrsLiveSource::on_meta_data[srs_app_source.cpp]
                  SrsMetaCache::update_data
      }
 
 
  
  ??
  SrsHls::hls_show_mux_log
```

坑s：

`clang: warning: argument unused during compilation: '-rdynamic' [-Wunused-command-line-argument]` 这个在编.o时没用， link时用，所以没事


## 拉rtmp

```
stream_service_cycle
  SrsRtmpServer::identify_client
    identify_play_client
  SrsRtmpServer::start_play
  SrsRtmpConn::playing
    SrsLiveSource::consumer_dumps
    do_playing
```
    
## 拉flv

```
SrsHttpConn::do_cycle
  SrsHttpConn::process_requests
    SrsHttpConn::process_request
      SrsHttpServer::serve_http
        SrsLiveStream::serve_http
          SrsLiveStream::do_serve_http
            create_consumer
srs_is_server_gracefully_close          
```

推流断开：

```
SrsServer::on_unpublish
  http_server->http_unmount(s, r)


SrsLiveStream::do_serve_http
   while (entry->enabled)
```

    
# log header:

```
│ 232     int written = -1;¬
│ 233     if (dangerous) {¬
│ 234         if (tag) {¬
│ 235             written = snprintf(buffer, size,¬
│ 236                 "[%d-%02d-%02d %02d:%02d:%02d.%03d][%s][%d][%s][%d][%s] ",¬
│ 237                 1900 + now.tm_year, 1 + now.tm_mon, now.tm_mday, now.tm_hour, now.tm_min, now.tm_sec, (int)(tv.tv_usec / 1000),¬
│ 238                 level, getpid(), cid.c_str(), errno, tag);¬
│ 239         } else {¬
│ 240             written = snprintf(buffer, size,¬
│ 241                 "[%d-%02d-%02d %02d:%02d:%02d.%03d][%s][%d][%s][%d] ",¬
│ 242                 1900 + now.tm_year, 1 + now.tm_mon, now.tm_mday, now.tm_hour, now.tm_min, now.tm_sec, (int)(tv.tv_usec / 1000),¬
│ 243                 level, getpid(), cid.c_str(), errno);¬
│ 244         }¬
│ 245     } else {¬
│ 246         if (tag) {¬
│ 247             written = snprintf(buffer, size,¬
│ 248                 "[%d-%02d-%02d %02d:%02d:%02d.%03d][%s][%d][%s][%s] ",¬
│ 249                 1900 + now.tm_year, 1 + now.tm_mon, now.tm_mday, now.tm_hour, now.tm_min, now.tm_sec, (int)(tv.tv_usec / 1000),¬
│ 250                 level, getpid(), cid.c_str(), tag);¬
│ 251         } else {¬
│ 252             written = snprintf(buffer, size,¬
│ 253                 "[%d-%02d-%02d %02d:%02d:%02d.%03d][%s][%d][%s] ",¬
│ 254                 1900 + now.tm_year, 1 + now.tm_mon, now.tm_mday, now.tm_hour, now.tm_min, now.tm_sec, (int)(tv.tv_usec / 1000),¬
│ 255                 level, getpid(), cid.c_str());¬
│ 256         }¬
│ 257     }¬
```

## api

```
main
  do_main
    run_directly_or_daemon
      run_hybrid_server
        _srs_hybrid->register_server(new SrsServerAdapter());
          SrsServerAdapter::run()
            SrsServer::http_handle()
              http_api_mux->handle("/api/v1/streams/", new SrsGoApiStreams()))
                SrsGoApiStreams::serve_http
                  stat->find_stream(sid)
                  stat->dumps_streams(data, start, count)
```        

在publish时写入：
```
SrsRtmpConn::acquire_publish
 SrsLiveSource::on_publish()
   stat->on_stream_publish(req, _source_id.c_str());
      create_stream(vhost, req);
        // create stream if not exists.
```      



# webrtc

1. http://47.92.231.11:1985/rtc/v1/play/

   trunk/src/app/srs_app_rtc_api.cpp
   
   code: 0   
   sdp: xxx   
   server: 'vid-xxxx"   
   sessionid: "xxxxx"   

```
SrsGoApiRtcPlay::do_serve_http


SrsUdpStreamListener::listen
 SrsUdpListener::cycle
  SrsRtcServer::on_udp_packet
   SrsRtcConnection::on_stun 
    SrsRtcConnection::update_sendonly_socket
    SrsRtcConnection::on_binding_request  

SrsDtlsImpl::state_trace

```

```
{"time":"2022-12-22T08:02:17.604+08:00", "level":"Trace", "pid":1, "cid":"977a7ly7", "message":"TCP: clear zombies=1 resources, conns=2, removing=0, unsubs=0"}
{"time":"2022-12-22T08:02:17.604+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"TCP: disposing #0 resource(HttpConn)(0x5652cc2459d0), conns=2, disposing=1, zombies=0"}
{"time":"2022-12-22T08:02:17.605+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"RTC: session address init 172.17.67.1:26814"}
{"time":"2022-12-22T08:02:17.606+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"RTC: session STUN done, waiting DTLS handshake."}
{"time":"2022-12-22T08:02:17.623+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"DTLS: State Passive RECV, done=0, arq=0/0, r0=1, r1=0, len=157, cnt=22, size=144, hs=1"}
{"time":"2022-12-22T08:02:17.623+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"DTLS: State Passive SEND, done=0, arq=0/0, r0=-1, r1=2, len=638, cnt=22, size=82, hs=2"}
{"time":"2022-12-22T08:02:17.641+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"DTLS: State Passive RECV, done=0, arq=0/0, r0=1, r1=0, len=577, cnt=22, size=299, hs=11"}
{"time":"2022-12-22T08:02:17.642+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"DTLS: State Passive SEND, done=1, arq=0/0, r0=1, r1=0, len=554, cnt=22, size=466, hs=4"}
{"time":"2022-12-22T08:02:17.642+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"RTC: DTLS handshake done."}
{"time":"2022-12-22T08:02:17.642+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"RTC: session pub=0, sub=1, to=30000ms connection established"}
{"time":"2022-12-22T08:02:17.642+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"RTC: Subscriber url=/live/lk established"}
{"time":"2022-12-22T08:02:17.642+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"create consumer, no gop cache"}
{"time":"2022-12-22T08:02:17.642+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"RTC: start play url=/live/lk, source_id=772ky644/t28308g9, realtime=1, mw_msgs=0"}
{"time":"2022-12-22T08:02:19.150+08:00", "level":"Trace", "pid":1, "cid":"b24518u9", "message":"RTC: NACK ARQ seq=10014, ssrc=10143, ts=20861100, count=1/1, 647 bytes"}
{"time":"2022-12-22T08:02:21.199+08:00", "level":"Trace", "pid":1, "cid":"08cp485l", "message":"RTC: Server conns=1, rpkts=(6,rtp:0,stun:1,rtcp:5), spkts=(364,rtp:364,stun:1,rtcp:0), rnk=(1,1,h:1,m:0), fid=(id:0,fid:6,ffid:0,addr:1,faddr:6)"}
{"time":"2022-12-22T08:02:21.762+08:00", "level":"Trace", "pid":1, "cid":"772ky644", "event
```


# 其他

* `make -j 8`  not `make -j8`


# 集群模式

* edge 配置 cluster origin 是用于forward lb的
* origin 的 coworker : rtmp 302
* 在play阶段，如果本地没流，会根据配置去coworker查是否有流，然后302

## k8s 


* pod 地址： `srs-origin-1.socs.NAMESPACE.svc.cluster.local.`  `POD.SERVICE.NS.svc.cluster.local.` = SERVICE
* ` docker run -v /Users/jinleileiking/visionular/wzsrs/trunk:/srstrunk --name srs  -P   -dit  srsdev /bin/bash`


## st

* http://state-threads.sourceforge.net/docs/reference.html
* https://coolshell.cn/articles/12012.html




# OBS

* 推时间流看延时：浏览器写time.is就行了
