
# ion-sfu 在B点publish后，怎么发的negotiation

sfu建立B的pc后，会调用 APc.Addtrack. 导致给APc发negotiation

```
0  0x0000000000897083 in github.com/pion/webrtc/v3.(*PeerConnection).onNegotiationNeeded
   at /home/jinlei1/ksyun/pkg/mod/github.com/pion/webrtc/v3@v3.0.0-beta.4.0.20200902134452-789ff0975342/peerconnection.go:257
1  0x00000000008a1024 in github.com/pion/webrtc/v3.(*PeerConnection).newRTPTransceiver
   at /home/jinlei1/ksyun/pkg/mod/github.com/pion/webrtc/v3@v3.0.0-beta.4.0.20200902134452-789ff0975342/peerconnection.go:1810
2  0x000000000089ff42 in github.com/pion/webrtc/v3.(*PeerConnection).AddTransceiverFromTrack
   at /home/jinlei1/ksyun/pkg/mod/github.com/pion/webrtc/v3@v3.0.0-beta.4.0.20200902134452-789ff0975342/peerconnection.go:1590
3  0x000000000089f20a in github.com/pion/webrtc/v3.(*PeerConnection).AddTrack
   at /home/jinlei1/ksyun/pkg/mod/github.com/pion/webrtc/v3@v3.0.0-beta.4.0.20200902134452-789ff0975342/peerconnection.go:1472
4  0x00000000009ecdaf in github.com/pion/ion-sfu/pkg.(*WebRTCTransport).NewSender
   at /home/jinlei1/ksyun/pkg/mod/github.com/pion/ion-sfu@v1.0.7/pkg/webrtctransport.go:230
5  0x00000000009ebd99 in github.com/pion/ion-sfu/pkg.NewWebRTCTransport
   at /home/jinlei1/ksyun/pkg/mod/github.com/pion/ion-sfu@v1.0.7/pkg/webrtctransport.go:57
6  0x00000000009eb7c0 in github.com/pion/ion-sfu/pkg.(*SFU).NewWebRTCTransport
   at /home/jinlei1/ksyun/pkg/mod/github.com/pion/ion-sfu@v1.0.7/pkg/sfu.go:127
7  0x00000000009efe61 in main.(*RPC).Handle
   at /home/jinlei1/ksyun/src/11LiveChat/main.go:180
8  0x00000000008c2907 in github.com/sourcegraph/jsonrpc2.(*Conn).readMessages
   at /home/jinlei1/ksyun/pkg/mod/github.com/sourcegraph/jsonrpc2@v0.0.0-20200429184054-15c2290dcb37/jsonrpc2.go:522
9  0x0000000000463b61 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:1373
```

# 交互情况

```
chrome -O-> (0504 5229 1614)  sfu 
chrome <-A-  sfu
            sfu <-O- (7051 4547 1795)  safari
            sfu -A-> (0504 5229)       safari
chrome <-O-(7051 4547) sfu
chrome -A-> sfu
```
