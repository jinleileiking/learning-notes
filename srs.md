代码基于：23730fa48387fc41d847e2ab69d65ee721676b8e  4.0release

推流：

```
SrsRtmpConn::cycle()[srs_app_rtmp_conn.cpp]
  SrsRtmpConn::do_cycle()
    rtmp->handshake()
      SrsComplexHandshake::handshake_with_client
    rtmp->connect_app(req) = SrsSimpleRtmpClient::connect_app()
    ??
    SrsBasicRtmpClient::recv_message[srs_service_rtmp_conn.cpp]
      SrsProtocol::recv_message[srs_rtmp_stack.cpp]
        SrsProtocol::on_recv_message
          print_debug_info
    SrsRtmpConn::service_cycle()[srs_app_rtmp_conn.cpp]
      while(true){
      SrsRtmpConn::stream_service_cycle
        SrsRtmpConn::publishing
          SrsRtmpConn::do_publishing
            SrsRtmpConn::handle_publish_message
              SrsRtmpConn::process_publish_message
                SrsLiveSource::on_meta_data[srs_app_source.cpp]
                  SrsMetaCache::update_data
      }
    
    
  ??
  SrsHlsController::on_publish
 
  
  ??
  SrsHls::hls_show_mux_log
```


坑s：

`clang: warning: argument unused during compilation: '-rdynamic' [-Wunused-command-line-argument]` 这个在编.o时没用， link时用，所以没事