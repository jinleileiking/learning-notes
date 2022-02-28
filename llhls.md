# refs

## the o

* https://www.theoplayer.com/blog/evolution-of-ll-hls
* https://www.theoplayer.com/blog/how-does-ll-hls-work
* https://www.theoplayer.com/blog/implementing-ll-hls-today
* https://www.theoplayer.com/blog/implementing-ll-hls-abr-subs-drm-ssai
* https://www.theoplayer.com/blog/optimizing-ll-hls-4-key-factors-affecting-its-performance
* https://www.theoplayer.com/blog/optimizing-ll-hls-the-impacts-of-gop-size-on-viewing-experience
* https://www.theoplayer.com/blog/optimizing-ll-hls-4-recommendations


## mux

```
#EXTM3U
#EXT-X-TARGETDURATION:4
#EXT-X-VERSION:6
#EXT-X-SERVER-CONTROL:CAN-BLOCK-RELOAD=YES,CAN-SKIP-UNTIL=24,PART-HOLD-BACK=3.012
#EXT-X-PART-INF:PART-TARGET=1.004000
#EXT-X-MEDIA-SEQUENCE:209179
#EXT-X-MAP:URI="fileSequence11.mp4"
#EXTINF:3.98933,
fileSequence209267.m4s
#EXTINF:3.98933,
fileSequence209268.m4s
#EXTINF:3.98933,
fileSequence209269.m4s
#EXTINF:3.98933,
fileSequence209270.m4s
#EXT-X-PROGRAM-DATE-TIME:2022-02-28T07:56:10.698Z
#EXTINF:3.98933,
fileSequence209271.m4s
#EXTINF:3.98933,
fileSequence209272.m4s
#EXTINF:3.98933,
fileSequence209273.m4s
#EXTINF:3.98933,
fileSequence209274.m4s
#EXTINF:3.98933,
fileSequence209275.m4s
#EXT-X-PROGRAM-DATE-TIME:2022-02-28T07:56:30.644Z
#EXTINF:3.98933,
fileSequence209276.m4s
#EXTINF:3.98933,
fileSequence209277.m4s
#EXTINF:3.98933,
fileSequence209278.m4s
#EXTINF:3.98933,
fileSequence209279.m4s
#EXTINF:3.98933,
fileSequence209280.m4s
#EXT-X-PROGRAM-DATE-TIME:2022-02-28T07:56:50.591Z
#EXT-X-PART:DURATION=1.00267,URI="lowLatencySeg.m4s?segment=filePart209281.1.m4s"
#EXT-X-PART:DURATION=1.00267,URI="lowLatencySeg.m4s?segment=filePart209281.2.m4s"
#EXT-X-PART:DURATION=1.00267,URI="lowLatencySeg.m4s?segment=filePart209281.3.m4s"
#EXT-X-PART:DURATION=0.98133,URI="lowLatencySeg.m4s?segment=filePart209281.4.m4s"
#EXTINF:3.98933,
fileSequence209281.m4s
#EXT-X-PART:DURATION=1.00267,URI="lowLatencySeg.m4s?segment=filePart209282.1.m4s"
#EXT-X-PART:DURATION=1.00267,URI="lowLatencySeg.m4s?segment=filePart209282.2.m4s"
#EXT-X-PART:DURATION=1.00267,URI="lowLatencySeg.m4s?segment=filePart209282.3.m4s"
#EXT-X-PART:DURATION=0.98133,URI="lowLatencySeg.m4s?segment=filePart209282.4.m4s"
#EXTINF:3.98933,
fileSequence209282.m4s
#EXT-X-PART:DURATION=1.00267,URI="lowLatencySeg.m4s?segment=filePart209283.1.m4s"
#EXT-X-PRELOAD-HINT:TYPE=PART,URI="lowLatencySeg.m4s?segment=filePart209283.2.m4s"
#EXT-X-RENDITION-REPORT:URI="../media0/lowLatencyHLS.m3u8",LAST-MSN=208636,LAST-PART=3
#EXT-X-RENDITION-REPORT:URI="../media1/lowLatencyHLS.m3u8",LAST-MSN=208636,LAST-PART=3
#EXT-X-RENDITION-REPORT:URI="../media2/lowLatencyHLS.m3u8",LAST-MSN=208636,LAST-PART=3
#EXT-X-RENDITION-REPORT:URI="../media3/lowLatencyHLS.m3u8",LAST-MSN=208636,LAST-PART=3
#
```

16个m4s，最后 2个拆part, 5个m4s加一个datetime
