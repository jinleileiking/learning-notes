# call ListenAndServeTLS

```
ListenAndServeTLS
  s.serveImpl
    quicListenAddr
      conn, err := net.ListenUDP("udp", udpAddr)¬
      listen
      = sessionHandler, err := getMultiplexer().AddConn(conn, config.ConnectionIDLength)
      = AddConn
        manager := m.newPacketHandlerManager(c, connIDLen, m.logger)
        = newPacketHandlerMap
          = go m.listen
     sess, err := ln.Accept()
     = (s *server) Acceppt
       select {  case sess = <-s.sessionQueue: 
     go s.handleHeaderStream(sess.(streamCreator))
```

```
- 38 func getMultiplexer() multiplexer {¬
| 39 ▸   connMuxerOnce.Do(func() {¬
| 40 ▸   ▸   connMuxer = &connMultiplexer{¬
| 41 ▸   ▸   ▸   conns:                   make(map[net.PacketConn]connManager),¬
| 42 ▸   ▸   ▸   logger:                  utils.DefaultLogger.WithPrefix("muxer"),¬
| 43 ▸   ▸   ▸   newPacketHandlerManager: newPacketHandlerMap,¬
| 44 ▸   ▸   }¬
| 45 ▸   })¬
| 46 ▸   return connMuxer¬
| 47 }¬
```

# Stream Flow

```
server —---------  cli
                 <- CHLO 1
REJ -> 1
ACK -> 2 (1-1)
                 <- ACk  2 (1-1)
                 <- CHLO 3
ACK -> 3 (1-3)
REJ -> 4 5 
                  <-ACK 4(1-5)
                  <-ConnectionClose 5
    
```

```
server —---------  cli
                 <- CHLO 1 (1072B)
REJ -> 1 (265B)
ACK -> 2 (1-1) // 我已经接收了你的序号1的包了
                 <- ACk  2 (1-1)
                 <- CHLO 3 (1074B)
ACK -> 3 (1-3)
REJ -> 4 (776B)
                  <-ACK 4 (1-4)
                  <-CHLO 5 (1074B)
SHLO -> 5 (222B)
                  <-Ping 6
ACK -> 6 (1-6)
                  <-"foobar"  7   streamid:3
"foobar" -> 7 
    
```





# Receive CHLO and sending REJ

pcap:

```
0000   ff 51 30 34 34 50 04 e5 1d 3a 07 44 d7 5e 00 00   ÿQ044P.å.:.D×^..
0010   00 01 9f 68 76 36 64 f0 cb 14 51 fe 38 55 80 01   ...hv6dðË.Qþ8U..
0020   43 48 4c 4f 09 00 00 00 50 41 44 00 8d 03 00 00   CHLO....PAD.....
0030   53 4e 49 00 98 03 00 00 56 45 52 00 9c 03 00 00   SNI.....VER.....
0040   43 43 53 00 ac 03 00 00 50 44 4d 44 b0 03 00 00   CCS.¬...PDMD°...
0050   49 43 53 4c b4 03 00 00 4d 49 44 53 b8 03 00 00   ICSL´...MIDS¸...
0060   43 46 43 57 bc 03 00 00 53 46 43 57 c0 03 00 00   CFCW¼...SFCWÀ...
0070   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
 
```

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+
   |1|1|T T|X X X X|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Version (32)                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |DCIL(4)|SCIL(4)|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |               Destination Connection ID (0/32..144)         ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                 Source Connection ID (0/32..144)            ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                    Figure 9: Long Header Packet Format
```

```
ff // Longheader  init packet
51 30 34 34 // version: Q44
50    // dest = 8 src = 0
04 e5 1d 3a 07 44 d7 5e // dest cid
00 00 00 01 // packet number = 1

9f 68 76 36 64 f0 cb 14 51 fe 38 55 // 12bytes ciphertext
80  // stream frame
01  // 000 xxx(hasoffset) finbit Datalenpresent 1 --- finbit: false, datalenPresent: false, hasoffset: 
43 48 4c 4f // message tag: CHLO 
09 00 00 00 // 9 pairs
50 41 44 00 // PAD
8d 03 00 00 // 0x38d
53 4e 49 00 // SNI
98 03 00 00 // 0x398 
56 45 52 00 // SNI
9c 03 00 00 // 0x39c
43 43 53 00 // CCS
ac 03 00 00 // 0x3ac
.... 


0x0410:  0000 0000 0000 0000 0031 302e 3634 2e37  .........10.64.7
```

log:

```
    Serving new connection: 0xfee35e4cdffbc452, version gQUIC 44 from xxxxxxxxx
    <- Reading packet 0x1 (1072 bytes) for connection 0xfee35e4cdffbc452, unencrypted
    server  Long Header{Type: Initial, DestConnectionID: 0xfee35e4cdffbc452, SrcConnectionID: (empty), PacketNumber: 0x1, PacketNumberLen: 4, Version: gQUIC 44}
    server  <- &wire.StreamFrame{StreamID: 1, FinBit: false, Offset: 0x0, Data length: 0x410, Offset + Data length: 0x410}
    server Got CHLO:
erver Got CHLO:
        SNI : "10.64.7.106"
        VER : "\x00\x00\x00\x00"
        CCS : "\x01\xe8\x81`\x92\x92\x1a\xe8~퀆\xa2\x15\x82\x91"
        PDMD: "X509"
        ICSL: "\x1e\x00\x00\x00"
        MIDS: "d\x00\x00\x00"
        CFCW: "\x00\xc0\x00\x00"
        SFCW: "\x00\x80\x00\x00"
        PAD : (909 bytes)
server Sending REJ :
        STK : "xxxxxxx"
        SVID: "quic-go"
        SCFG: "SCFG          xxxxxxxxxxxxx"

server Sending REJ :
        STK : "xxxxxxxxxxx"
        SVID: "quic-go"
        PROF: "xxxxxxxxxxxx"
        SCFG: "SCFG xxxxxxxxxxxx"
        CRT: "xxxxxxxxxxxxxxxx#"
```

**Call Path**

分析packet

```
m.listen()
  h.conn.ReadFrom(data)
  (h *packetHandlerMap) handlePacket  [packet_handler_map.go]
    wire.ParseInvariantHeader[internal/vire/header_parser.go]  --- 分析packet header, 打印header
    handler, ok := h.handlers[string(iHdr.DestConnectionID)]
      if !ok {
          handlePacket = server.handlePacket //当initial packet来时，走这，采用server的handlePack
      } else { // 找到了
          version = handler.GetVersion()
          handlePacket = handler.handlePacket // 采用这个connection的handler
      }
    func (s *server) handlePacket(p *receivedPacket) [server.go]
      server.handlePacketImpl
         s.newSession ---- 收到字节后，建立一个session
           s.preSetup()
            s.cryptoStream = s.newCryptoStream()
              newCryptoStream
                newStream(, sender,)
           cs, err := newCryptoSetup(s.cryptoStream
         s.sessionHandler.Add(hdr.DestConnectionID, newServerSession(sess, s.config, s.logger))  // 这里有server启动一个server session
         = (h *packetHandlerMap) Add // 将server session的 handler 给map
         go sess.run()
         sess.handlePacket(p) //这里不会调用impl
         = (s *session) handlePacket(p *receivedPacket) // 这里调用的是 session的handle
             select {¬ case s.receivedPackets <- p: //将数据发给channel
             
```

分析stream

```
 (s *session) run()
 
    go func() {¬ if err := s.cryptoStreamHandler.HandleCryptoStream(); err != nil {¬
 
    for { select { 
    case p := <-s.receivedPackets:
    (s *session) handlePacketImpl(p *receivedPacket) [session.go] 
      packet, err := s.unpacker.Unpack(hdr.Raw, hdr, p.data) --- 解密在这里
      = (u *packetUnpackerGQUIC) Unpack [packet_unpacker.go] -- 这里会分析stream 生成stream数组
       decrypted, encryptionLevel, err := u.aead.Open(data[:0], data, hdr.PacketNumber, headerBinary)
       =  (h *cryptoSetupServer) Open
         h.nullAEAD.Open
         = func (n *nullAEADFNV128a) Open [internal/crypto/null_aead_fnv128a.go]
      s.handleFrames
        s.handleStreamFrame
          str.handleStreamFrame
    
```

callback stack:

```
0  0x00000000007d111b in github.com/lucas-clemente/quic-go.(*server).handlePacketImpl
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/server.go:353
1  0x00000000007d0fc9 in github.com/lucas-clemente/quic-go.(*server).handlePacket
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/server.go:348
2  0x00000000007e4c39 in github.com/lucas-clemente/quic-go.unknownPacketHandler.handlePacket-fm
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/server.go:29
3  0x00000000007c5087 in github.com/lucas-clemente/quic-go.(*packetHandlerMap).handlePacket
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/packet_handler_map.go:191
4  0x00000000007c4947 in github.com/lucas-clemente/quic-go.(*packetHandlerMap).listen
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/packet_handler_map.go:135
5  0x000000000045cb81 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
```





```
0  0x00000000007baba3 in github.com/lucas-clemente/quic-go/internal/ackhandler.(*receivedPacketHandler).maybeQueueAck
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/internal/ackhandler/received_packet_handler.go:121
1  0x00000000007ba8d6 in github.com/lucas-clemente/quic-go/internal/ackhandler.(*receivedPacketHandler).ReceivedPacket
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/internal/ackhandler/received_packet_handler.go:85
2  0x00000000007d725a in github.com/lucas-clemente/quic-go.(*session).handlePacketImpl
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:688
3  0x00000000007d60e1 in github.com/lucas-clemente/quic-go.(*session).run
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:484
4  0x000000000045cb81 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
```



## CHLO

call stack:

```
session.run()
  s.cryptoStreamHandler.HandleCryptoStream();
  = HandleCryptoStream [crypto_setup_server.go]
    ParseHandshakeMessage(io.TeeReader(h.cryptoStream, &chloData))   --- h.cryptoStream什么时候建的?注意这个是cryptoSetupServer的stream，不是 cryptostreamhandler的stream
    h.handleMessage
      h.handleInchoateCHLO
      h.cryptoStream.Write(reply)

```



backtrace:

```
0  0x000000000079f97b in github.com/lucas-clemente/quic-go/internal/handshake.(*cryptoSetupServer).handleInchoateCHLO
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/internal/handshake/crypto_setup_server.go:306
1  0x000000000079e432 in github.com/lucas-clemente/quic-go/internal/handshake.(*cryptoSetupServer).handleMessage
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/internal/handshake/crypto_setup_server.go:192
2  0x000000000079e09a in github.com/lucas-clemente/quic-go/internal/handshake.(*cryptoSetupServer).HandleCryptoStream
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/internal/handshake/crypto_setup_server.go:119
3  0x00000000007e1f6a in github.com/lucas-clemente/quic-go.(*session).run.func1
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:451
4  0x000000000045cb81 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
```





# send first ack

calltrace:

```
(s *session) run() 
  
```





callstack

```
(dlv) bt
0  0x0000000000902ea6 in github.com/lucas-clemente/quic-go/internal/wire.LogFrame
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/internal/wire/log.go:29
1  0x000000000093e848 in github.com/lucas-clemente/quic-go.(*session).logPacket
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:1106
2  0x000000000093e1cb in github.com/lucas-clemente/quic-go.(*session).sendPackedPacket
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:1082
3  0x000000000093e0c0 in github.com/lucas-clemente/quic-go.(*session).sendPacket
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:1074
4  0x000000000093ce58 in github.com/lucas-clemente/quic-go.(*session).sendPackets
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:960
5  0x0000000000938020 in github.com/lucas-clemente/quic-go.(*session).run
   at /home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:543
6  0x0000000000464c51 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
```







# receive ACK

log:

```
server <- Reading packet 0x4 (36 bytes) for connection 0x94a776694f26e485, unencrypted
server  Long Header{Type: Handshake, DestConnectionID: 0x94a776694f26e485, SrcConnectionID: (empty), PacketNumber: 0x4, PacketNumberLen: 4, Version: gQUIC 44}
server  <- &wire.AckFrame{LargestAcked: 0x5, LowestAcked: 0x1, DelayTime: 41µs}
server  updated RTT: 457µs (σ: 232µs)
server  newly acked packets (2): [0x4 0x5]
server <- Reading packet 0x5 (37 bytes) for connection 0x94a776694f26e485, unencrypted
server  Long Header{Type: Handshake, DestConnectionID: 0x94a776694f26e485, SrcConnectionID: (empty), PacketNumber: 0x5, PacketNumberLen: 4, Version: gQUIC 44}
server  Setting ACK timer to max ack delay: 25ms
```







ln.Accept()

```
- 296 func (s *server) Accept() (Session, error) {¬
| 297 ▸   var sess Session¬
| 298 ▸   select {¬
| 299 ▸   case sess = <-s.sessionQueue:¬
| 300 ▸   ▸   return sess, nil¬
| 301 ▸   case <-s.errorChan:¬
| 302 ▸   ▸   return nil, s.serverError¬
| 303 ▸   }¬
| 304 }¬
```





# Send

backtrace:

```
 0  0x0000000000623ea3 in github.com/lucas-clemente/quic-go.newStreamsMapLegacy
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/streams_map_legacy.go:42
 1  0x00000000006274fc in github.com/lucas-clemente/quic-go.glob..func2
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:278
 2  0x0000000000606e7a in github.com/lucas-clemente/quic-go.(*client).createNewGQUICSession
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/client.go:506
 3  0x0000000000604e9f in github.com/lucas-clemente/quic-go.(*client).dialGQUIC
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/client.go:296
 4  0x0000000000604d6b in github.com/lucas-clemente/quic-go.(*client).dial
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/client.go:290
 5  0x0000000000603f1e in github.com/lucas-clemente/quic-go.dialContext
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/client.go:144
 6  0x0000000000603c1a in github.com/lucas-clemente/quic-go.DialAddrContext
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/client.go:90
 7  0x000000000062d85c in github.com/lucas-clemente/quic-go.DialAddr
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/client.go:71
 8  0x000000000062d85c in main.clientMain
```





newStreamsMapLegacy

  

```go
      if pers == protocol.PerspectiveServer {
          sm.nextStreamToOpen = nextServerInitiatedStream
          sm.nextStreamToAccept = nextClientInitiatedStream
      } else {
          sm.nextStreamToOpen = nextClientInitiatedStream
          sm.nextStreamToAccept = nextServerInitiatedStream
      }
```







> stream, err := session.OpenStreamSync()

```
=  func (s *session) OpenStreamSync()

=  func (m *streamsMapLegacy) OpenStreamSync()

  openStreamImpl
    s := m.newStream(m.nextStreamToOpen)

= func (s *session) newStream(id protocol.StreamID) streamI


```





# Flow control when receive stream frame

code flow

```
(s *session) handlePacketImpl
  (s *session) handleFrames
    (s *session) handleStreamFrame
      s.streamsMap.GetOrOpenReceiveStream =  (m *streamsMapLegacy) GetOrOpenReceiveStream
        (m *streamsMapLegacy) getOrOpenStream
         m.openRemoteStream(sid)
            m.newStream(id) = (s *session) newStream
              s.newFlowController(id)
```





# 

```
​```
```



bt

```
 0  0x00000000006053d3 in github.com/lucas-clemente/quic-go/internal/protocol.VersionNumber.StreamContributesToConnectionFlowControl
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/internal/protocol/version.go:110
 1  0x000000000070b4ff in github.com/lucas-clemente/quic-go.(*session).newFlowController
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:1163
 2  0x000000000070b3b4 in github.com/lucas-clemente/quic-go.(*session).newStream
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:1152
 3  0x000000000071b521 in github.com/lucas-clemente/quic-go.(*session).newStream-fm
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:1151
 4  0x0000000000711c45 in github.com/lucas-clemente/quic-go.(*streamsMapLegacy).openRemoteStream
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/streams_map_legacy.go:145
 5  0x0000000000711827 in github.com/lucas-clemente/quic-go.(*streamsMapLegacy).getOrOpenStream
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/streams_map_legacy.go:120
 6  0x0000000000711155 in github.com/lucas-clemente/quic-go.(*streamsMapLegacy).GetOrOpenReceiveStream
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/streams_map_legacy.go:82
 7  0x0000000000707983 in github.com/lucas-clemente/quic-go.(*session).handleStreamFrame
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:759
 8  0x00000000007070e4 in github.com/lucas-clemente/quic-go.(*session).handleFrames
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:702
 9  0x000000000070661c in github.com/lucas-clemente/quic-go.(*session).handlePacketImpl
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:693
10  0x00000000007048f0 in github.com/lucas-clemente/quic-go.(*session).run
    at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:484
11  0x00000000004601d1 in runtime.goexit
    at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
(dlv)
```







StreamContributesToConnectionFlowControl 对 stream id 进行检查 id = 3 则不进行update window







bt

```
(s *session) run()
  (s *session) sendPackets()
    case ackhandler.SendAny:
      (s *session) sendPacket()
          s.windowUpdateQueue.QueueAll()
```





```
0  0x000000000071504b in github.com/lucas-clemente/quic-go.(*windowUpdateQueue).QueueAll
   at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/window_update_queue.go:47
1  0x000000000070a41a in github.com/lucas-clemente/quic-go.(*session).sendPacket
   at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:1067
2  0x00000000007092a8 in github.com/lucas-clemente/quic-go.(*session).sendPackets
   at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:960
3  0x0000000000704556 in github.com/lucas-clemente/quic-go.(*session).run
   at /home/jinlei1/ksyun/pkg/mod/github.com/lucas-clemente/quic-go@v0.10.2/session.go:543
4  0x00000000004601d1 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
```





### Sending UpdateWindow

```
goroutine 9 [running]:
runtime/debug.Stack(0x5f88b6, 0x9683c0, 0xc0001bcdb0)
        /home/jinlei1/os/go/src/runtime/debug/stack.go:24 +0x9d
runtime/debug.PrintStack()
        /home/jinlei1/os/go/src/runtime/debug/stack.go:16 +0x22
github.com/lucas-clemente/quic-go/internal/wire.(*MaxStreamDataFrame).Write(0xc0001bedc0, 0xc0001bcdb0, 0x51303434, 0x0, 0x0)
        /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/internal/wire/max_stream_data_frame.go:42 +0x26
github.com/lucas-clemente/quic-go.(*packetPackerLegacy).writeAndSealPacket(0xc0000b5040, 0xc0001cd860, 0xc00019ad40, 0x3, 0x4, 0x7fbbb9cb8630, 0xc0000c2c30, 0x0, 0x0, 0x0, ...)
        /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/packet_packer_legacy.go:421 +0x178
github.com/lucas-clemente/quic-go.(*packetPackerLegacy).PackPacket(0xc0000b5040, 0xc0000efc00, 0x0, 0xc0001a6000)
        /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/packet_packer_legacy.go:288 +0x24c
github.com/lucas-clemente/quic-go.(*session).sendPacket(0xc0000c66c0, 0xf, 0xc00019ad00, 0x0)
        /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:1070 +0x80
github.com/lucas-clemente/quic-go.(*session).sendPackets(0xc0000c66c0, 0x11c764eb3, 0x94be40)
        /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:961 +0x148
github.com/lucas-clemente/quic-go.(*session).run(0xc0000c66c0, 0x0, 0x0)
        /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:544 +0x65a
created by github.com/lucas-clemente/quic-go.(*server).handlePacketImpl
        /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/server.go:402 +0x514
2019/07/24 15:05:27 server -> Sending packet 0xd (1252 bytes) for connection 0x40cccca371a71b48, forward-secure
2019/07/24 15:05:27 server      Short Header{DestConnectionID: (empty), PacketNumber: 0xd, PacketNumberLen: 2, KeyPhase: 0}
2019/07/24 15:05:27 server      -> &wire.AckFrame{LargestAcked: 0xe, LowestAcked: 0x9, DelayTime: 923.847µs}
2019/07/24 15:05:27 server      -> &wire.MaxStreamDataFrame{StreamID:0x3, ByteOffset:0xa17b}
2019/07/24 15:05:27 server      -> &wire.StreamFrame{StreamID: 3, FinBit: false, Offset: 0x1cb8, Data length: 0x4be, Offset + Data length: 0x2176}
```



# Send Stream Frame

```
0  0x000000000062db3f in github.com/lucas-clemente/quic-go.(*framer).QueueControlFrame-fm
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/framer.go:36
1  0x0000000000628a6e in github.com/lucas-clemente/quic-go.(*windowUpdateQueue).QueueAll
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/window_update_queue.go:67
2  0x00000000006211d3 in github.com/lucas-clemente/quic-go.(*session).sendPacket
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:1068
3  0x0000000000620658 in github.com/lucas-clemente/quic-go.(*session).sendPackets
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:961
4  0x000000000061ce4a in github.com/lucas-clemente/quic-go.(*session).run
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:544
5  0x00000000004587f1 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
```



callstack

```
0  0x000000000062db3f in github.com/lucas-clemente/quic-go.(*framer).QueueControlFrame-fm
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/framer.go:36
1  0x0000000000628a6e in github.com/lucas-clemente/quic-go.(*windowUpdateQueue).QueueAll
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/window_update_queue.go:67
2  0x00000000006211d3 in github.com/lucas-clemente/quic-go.(*session).sendPacket
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:1068
3  0x0000000000620658 in github.com/lucas-clemente/quic-go.(*session).sendPackets
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:961
4  0x000000000061ce4a in github.com/lucas-clemente/quic-go.(*session).run
   at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:544
5  0x00000000004587f1 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
```









```
(s *receiveStream) readImpl
          if s.streamID != s.version.CryptoStreamID() {¬
              s.flowController.MaybeQueueWindowUpdate()¬
           }¬
           
           
      func (c *streamFlowController) MaybeQueueWindowUpdate() {
        c.mutex.Lock()
        hasWindowUpdate := !c.receivedFinalOffset && c.hasWindowUpdate()
        c.mutex.Unlock()
        if hasWindowUpdate {
            c.queueWindowUpdate() = onHasStreamWindowUpdate
        }
        if c.contributesToConnection {
            c.connection.MaybeQueueWindowUpdate()
        }
      }
    
    
      func (c *connectionFlowController) MaybeQueueWindowUpdate() {
      c.mutex.Lock()
      hasWindowUpdate := c.hasWindowUpdate()
      c.mutex.Unlock()
      if hasWindowUpdate {
          c.queueWindowUpdate() = onHasConnectionWindowUpdate
      }
  }
  
   func (s *session) onHasStreamWindowUpdate(id protocol.StreamID) {
        s.windowUpdateQueue.AddStream(id)
        s.scheduleSending()
    }
    
    
    func (s *session) onHasConnectionWindowUpdate() {
        s.windowUpdateQueue.AddConnection()
        s.scheduleSending()
    }
```



# Client sending Block

```
0  0x0000000000631b83 in github.com/lucas-clemente/quic-go.(*framer).QueueControlFrame
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/framer.go:36
 1  0x000000000064a286 in github.com/lucas-clemente/quic-go.(*session).queueControlFrame
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:1231
 2  0x000000000064a5e7 in github.com/lucas-clemente/quic-go.(*uniStreamSender).queueControlFrame
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/stream.go:34
 3  0x000000000063d90f in github.com/lucas-clemente/quic-go.(*sendStream).popStreamFrameImpl
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/send_stream.go:183
 4  0x000000000063d6b5 in github.com/lucas-clemente/quic-go.(*sendStream).popStreamFrame
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/send_stream.go:149
 5  0x00000000006321e3 in github.com/lucas-clemente/quic-go.(*framer).AppendStreamFrames
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/framer.go:89
 6  0x0000000000639ce9 in github.com/lucas-clemente/quic-go.(*packetPackerLegacy).composeNextPacket
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/packet_packer_legacy.go:363
 7  0x000000000063939c in github.com/lucas-clemente/quic-go.(*packetPackerLegacy).PackPacket
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/packet_packer_legacy.go:267
 8  0x0000000000648d90 in github.com/lucas-clemente/quic-go.(*session).sendPacket
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:1071
 9  0x00000000006481f8 in github.com/lucas-clemente/quic-go.(*session).sendPackets
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:962
10  0x00000000006449ea in github.com/lucas-clemente/quic-go.(*session).run
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/session.go:545
11  0x00000000006522a1 in github.com/lucas-clemente/quic-go.(*client).establishSecureConnection.func1
    at /data/home/jinlei1/ksyun/src/github.com/lucas-clemente/quic-go/client.go:344
12  0x0000000000458811 in runtime.goexit
    at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1337
```

```
 0  0x0000000000631b83 in github.com/lucas-clemente/quic-go.(*framer).QueueControlFrame
 1  0x000000000064a286 in github.com/lucas-clemente/quic-go.(*session).queueControlFrame
 3  0x000000000063d90f in github.com/lucas-clemente/quic-go.(*sendStream).popStreamFrameImpl
 4  0x000000000063d6b5 in github.com/lucas-clemente/quic-go.(*sendStream).popStreamFrame
 5  0x00000000006321e3 in github.com/lucas-clemente/quic-go.(*framer).AppendStreamFrames
 6  0x0000000000639ce9 in github.com/lucas-clemente/quic-go.(*packetPackerLegacy).composeNextPacket
 7  0x000000000063939c in github.com/lucas-clemente/quic-go.(*packetPackerLegacy).PackPacket
 8  0x0000000000648d90 in github.com/lucas-clemente/quic-go.(*session).sendPacket
 9  0x00000000006481f8 in github.com/lucas-clemente/quic-go.(*session).sendPackets
10  0x00000000006449ea in github.com/lucas-clemente/quic-go.(*session).run
11  0x00000000006522a1 in github.com/lucas-clemente/quic-go.(*client).establishSecureConnection.func1
12  0x0000000000458811 in runtime.goexit
 
```



Window Update 发送时机：

- 接收包，比较接收窗口
- 快满了，发送一个，增大接收窗口
- 对端接收到Window update，增大发送窗口

Blocked 发送时机：

- 要发送包，对比窗口大小和已发数据量
- 如果窗口满了，则发送





为什么server不调用read，会导致发送block？

1. 不掉read，不会触发MayWindowUpdate
2. 不会发送WindowUpdate
3. 导致发送端窗口满，block了
