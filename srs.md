代码基于：23730fa48387fc41d847e2ab69d65ee721676b8e  4.0release

推流：

```
SrsRtmpConn::do_cycle()
  rtmp->handshake()
    SrsComplexHandshake::handshake_with_client
  rtmp->connect_app(req) = SrsSimpleRtmpClient::connect_app()
  ??
  SrsProtocol::recv_message
    SrsProtocol::on_recv_message
      print_debug_info
  SrsRtmpConn::service_cycle()
    SrsRtmpConn::stream_service_cycle
    
    
  ??
  SrsHlsController::on_publish
  
  ??
  SrsRtmpConn::do_publishing
  
  ??
  SrsMetaCache::update_data
  
  ??
  SrsHls::hls_show_mux_log
```
