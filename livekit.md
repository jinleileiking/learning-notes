# ingress

```mermaid
graph LR;
  pub-->|rtmp|go-rtmp
  gstreamer-->|webrtc|livekit;
  subgraph ingress
  go-rtmp-->|io.copy|relay-->|flv localhost:9090|gstreamer
  end
```
