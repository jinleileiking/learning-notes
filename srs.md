# 测试

srs单测基于c的测试，代码在src/utest

集成测试基于go test，代码在`3rdparty/srs-bench/*_test.go`






代码基于：23730fa48387fc41d847e2ab69d65ee721676b8e  4.0release

启动：

```
SrsServer::listen()
  SrsBufferListener::listen
  

```


推流：

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


# 其他

* `make -j 8`  not `make -j8`


## k8s 


* pod 地址： `srs-origin-1.socs.NAMESPACE.svc.cluster.local.`  `POD.SERVICE.NS.svc.cluster.local.` = SERVICE



## st

* http://state-threads.sourceforge.net/docs/reference.html
* https://coolshell.cn/articles/12012.html
