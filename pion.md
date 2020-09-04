# SFU 

> comment the `pingallcandidates`, sfu runs well.

data flow:



## When call `curl`, the server will:

### webrtc.New

the logs:
```
ice WARNING: 2019/02/18 15:40:58 could not listen udp xxx
ice WARNING: 2019/02/18 15:40:58 could not listen udp xxx
ice WARNING: 2019/02/18 15:40:58 could not listen udp xxx
ice WARNING: 2019/02/18 15:40:58 could not allocate udp6 stun:stun.l.google.com:19302: Failed to create STUN client: dial udp6 xxxxxxxxxxx  connect: network is unreachable
```


 
go program callback  trace:

```
0  0x000000000077b5db in github.com/pions/webrtc/pkg/ice.(*Agent).gatherCandidatesLocal
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/pkg/ice/agent.go:196
1  0x000000000077b005 in github.com/pions/webrtc/pkg/ice.NewAgent
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/pkg/ice/agent.go:159
2  0x0000000000868dee in github.com/pions/webrtc.(*RTCIceGatherer).Gather
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcicegatherer.go:67
3  0x000000000086cfbc in github.com/pions/webrtc.(*RTCPeerConnection).gather
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:482
4  0x000000000086b9d4 in github.com/pions/webrtc.(*API).NewRTCPeerConnection
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:159
5  0x000000000086154d in github.com/pions/webrtc.New
```

NewAgent will 

* `a.gatherCandidatesLocal` will listen udp ports. Each port will setup a goroutine to recv udp messages. 
    * The listenUDP will check from port = 0 to port = max, listen a port.
* setup a goroutine `a.taskLoop`. Then wait for pinging the all candidates.
```
| 312 ▸   ▸   // TODO this should be dynamic, and grow when the connection is stable¬
| 313 ▸   ▸   t := time.NewTicker(taskLoopInterval)¬
| 314 ▸   ▸   agent.connectivityTicker = t¬
| 315 ▸   ▸   agent.connectivityChan = t.C¬
```

* startConnectivityChecks will setup a timer. When fire (SetRemoteDescription), call the pingAllcandidates


### peerConnection.SetRemoteDescription

```
pc INFO: 2019/02/18 15:40:58 signaling state changed to have-remote-offer
ice DEBUG: 15:40:58.539395 agent.go:305: Started agent: isControlling? false, remoteUfrag: "ScIg", remotePwd: "TnmN1vmivhi71I6koIojEkSF"
pc INFO: 2019/02/18 15:40:58 ICE connection state changed: Checking
pc INFO: 2019/02/18 15:40:58 signaling state changed to stable
```

sfu will use the SDP from browser offer call this func


* `peerConnection.SetRemoteDescription` will set the state to `have-remote-offer` and run a goroutine `err := pc.iceTransport.Start`
    * ice start will run a goroutine for ice things:
        1. pc.iceTransport.Start will -> agent.Accept -> a.connect -> select -> wait for  ice connected (start session button) 
        2. dtls 
        3. call openSRTP hang up  at `<-receiver.Receive(RTCRtpReceiveParameters)` wait for `pc.onTrack`
* add remote candidates from SDP to icetransport
* startConnectivityChecks

```
0  0x000000000077d89b in github.com/pions/webrtc/pkg/ice.(*Agent).startConnectivityChecks
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/pkg/ice/agent.go:297
1  0x0000000000786153 in github.com/pions/webrtc/pkg/ice.(*Agent).connect
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/pkg/ice/transport.go:38
2  0x0000000000785f5b in github.com/pions/webrtc/pkg/ice.(*Agent).Accept
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/pkg/ice/transport.go:21
3  0x000000000086a9b3 in github.com/pions/webrtc.(*RTCIceTransport).Start
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcicetransport.go:95
4  0x00000000008796e8 in github.com/pions/webrtc.(*RTCPeerConnection).SetRemoteDescription.func2
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:805
```

```
0  0x000000000086c688 in github.com/pions/webrtc.(*RTCPeerConnection).onSignalingStateChange
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:245
1  0x000000000086e78e in github.com/pions/webrtc.(*RTCPeerConnection).setDescription
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:673
2  0x000000000086ff77 in github.com/pions/webrtc.(*RTCPeerConnection).SetRemoteDescription
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:732
```

call `onICEConnectionStateChange` when hooked
 
NewRTCPeerConnection -> pc.createICETransport() -> t.OnConnectionStateChange will setup `t.onConnectionStateChangeHdlr`

```
0  0x000000000086cbb8 in github.com/pions/webrtc.(*RTCPeerConnection).onICEConnectionStateChange
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:309
1  0x0000000000872df8 in github.com/pions/webrtc.(*RTCPeerConnection).iceStateChange
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:1405
2  0x0000000000879426 in github.com/pions/webrtc.(*RTCPeerConnection).createICETransport.func1
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:492
3  0x000000000086acac in github.com/pions/webrtc.(*RTCIceTransport).onConnectionStateChange
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcicetransport.go:140
4  0x0000000000879296 in github.com/pions/webrtc.(*RTCIceTransport).Start.func1
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcicetransport.go:71
```

 
pingCandidate will send STUN binding request to candidates.



## When click the `start session` button

My browser is in NAT. So you can see your ip is not your laptop ip in logs.

chrome will send bind request to sfu, sfu will recv udp 

`(c *Candidate) recvLoop()` -> is stun -> k `func (a *Agent) handleInbound` -> setValidPair -> ice to connected -> `Agent.taskLoop` -> checkKeepalive  

### on track 

as above said:

    * ice start will run a goroutine for ice things:
        1. pc.iceTransport.Start will -> agent.Accept -> a.connect -> select -> wait for  ice connected (start session button) 
        2. dtls 
        3. call openSRTP hang up  at `<-receiver.Receive(RTCRtpReceiveParameters)` wait for `pc.onTrack`
        4. call pc.onTrackHandler ---> that is what you set in example.



```
0  0x000000000086c4d3 in github.com/pions/webrtc.(*RTCPeerConnection).onTrack
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:281
1  0x0000000000879e3e in github.com/pions/webrtc.(*RTCPeerConnection).openSRTP.func1
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:932
2  0x000000000045d601 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:2361
```


```
0  0x000000000087086b in github.com/pions/webrtc.(*RTCPeerConnection).openSRTP
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:878
1  0x0000000000879624 in github.com/pions/webrtc.(*RTCPeerConnection).SetRemoteDescription.func2
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:833
2  0x000000000045d601 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:2361
```

```
0  0x000000000078166b in github.com/pions/webrtc/pkg/ice.(*Agent).handleInbound
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/pkg/ice/agent.go:704
1  0x000000000078a077 in github.com/pions/webrtc/pkg/ice.(*Candidate).recvLoop.func2
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/pkg/ice/candidate.go:149
2  0x000000000077f49a in github.com/pions/webrtc/pkg/ice.(*Agent).taskLoop
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/pkg/ice/agent.go:440
```


### logs

```
ice TRACE: 14:28:33.491539 agent.go:707: inbound STUN from xxxxx:21122 to hostxxx:9494
ice DEBUG: 14:28:33.491568 agent.go:710: detected a new peer-reflexive candiate: x
ice TRACE: 14:28:33.592461 agent.go:707: inbound STUN from x to srflx x related x
ice DEBUG: 14:28:33.592477 agent.go:710: detected a new peer-reflexive candiate: x
ice TRACE: 14:28:33.889331 agent.go:707: inbound STUN from x to host x
ice TRACE: 14:28:33.889374 agent.go:637: got controlled message (success? false, usepair? true)
ice TRACE: 14:28:33.889437 agent.go:389: Found valid candidate pair: prio 121792 (local, prio 3776) host x <-> prflx x related :0 (remote, prio 3360) (selected? true)
pc INFO: 2019/02/26 14:28:33 ICE connection state changed: Connected
ice TRACE: 14:28:33.939283 agent.go:707: inbound STUN from x to host x
ice TRACE: 14:28:33.939304 agent.go:637: got controlled message (success? false, usepair? true)
ice TRACE: 14:28:33.939335 agent.go:389: Found valid candidate pair: prio 121792 (local, prio 3776) host x <-> prflx x related :0 (remote, prio 3360) (selected? true)
ice TRACE: 14:28:34.816842 agent.go:431: checking keepalive
pc DEBUG: 14:28:34.948918 rtcpeerconnection.go:286: got new track: &{isRawRTP:false sampleInput:<nil> rawInput:<nil> rtcpInput:<nil> ID: PayloadType:96 Kind:video Label: Ssrc:1847792403 Codec:0xc42019c000 Packets:0xc4202665a0 RTCPPackets:0x
c420266600 Samples:<nil> RawRTP:<nil>}

Curl an base64 SDP to start sendonly peer connection
ice TRACE: 14:28:35.031824 agent.go:707: inbound STUN from x to host x
ice TRACE: 14:28:35.031844 agent.go:637: got controlled message (success? false, usepair? true)
ice TRACE: 14:28:35.031875 agent.go:389: Found valid candidate pair: prio 121792 (local, prio 3776) host x <-> prflx x related :0 (remote, prio 3360) (selected? true)
pc WARNING: 2019/02/26 14:28:35 Failed to unmarshal RTCP packet, discarding: rtcp: packet too short
ice TRACE: 14:28:36.037293 agent.go:707: inbound STUN from x to host x
ice TRACE: 14:28:36.037323 agent.go:637: got controlled message (success? false, usepair? true)
ice TRACE: 14:28:36.037360 agent.go:389: Found valid candidate pair: prio 121792 (local, prio 3776) host x <-> prflx x related :0 (remote, prio 3360) (selected? true)
pc WARNING: 2019/02/26 14:28:36 Failed to unmarshal RTCP packet, discarding: rtcp: packet too short
```

### receive rtcp

Recevie will setup up two goroutine. One for RTP, One for Rtcp.

```
0  0x00000000008a71c3 in github.com/pions/webrtc.(*RTCRtpReceiver).Receive
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcrtpreceiver.go:51
1  0x00000000008ac04a in github.com/pions/webrtc.(*RTCPeerConnection).openSRTP.func1
   at /home/jinlei1/ksyun/src/github.com/pions/webrtc/rtcpeerconnection.go:908
2  0x000000000045d601 in runtime.goexit
   at /home/jinlei1/os/go/src/runtime/asm_amd64.s:2361
```
