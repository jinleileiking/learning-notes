* 主M3u8-> 选第一个level -> 子M3u8 -> 

```
#EXT-X-PROGRAM-DATE-TIME:2022-05-11T02:35:53.500+0000
#EXT-X-PART:DURATION=1.00000,INDEPENDENT=YES,URI="playlistsegment2459.1652231625.m4s.part0.m4s"
#EXT-X-PART:DURATION=1.00000,URI="playlistsegment2459.1652231625.m4s.part1.m4s"
#EXTINF:2.00000,
playlistsegment2459.1652231625.m4s
#EXT-X-PART:DURATION=1.00000,INDEPENDENT=YES,URI="playlistsegment2460.1652231625.m4s.part0.m4s"
#EXT-X-PRELOAD-HINT:TYPE=PART,URI="playlistsegment2460.1652231625.m4s.part1.m4s"
#EXT-X-RENDITION-REPORT:URI="../1.2M/playlist.m3u8",LAST-MSN=2460,LAST-PART=0
#EXT-X-RENDITION-REPORT:URI="../700K/playlist.m3u8",LAST-MSN=2460,LAST-PART=0
```

* -> 以子m3u8最后一个part发起 m3u8请求，上面这个则： msn 2459, part 1



?????
======

```
loadSource
   this.trigger(Events.MANIFEST_LOADING
``` 
 
MANIFEST_LOADING 主m3u8 下载
========

```
class PlaylistLoader
  onManifestLoading
    this.load = 
  load  
  [XHR]
  loadsuccess
    handleMasterPlaylist
      hls.trigger(Events.MANIFEST_LOADED
```      
      
MANIFEST_LOADED 主m3u8 下载完
======

```
class LevelController
  onManifestLoaded ------> 主m3u8
    this.hls.trigger(Events.MANIFEST_PARSED
    this.hls.startLoad
      this.networkControllers.forEach((controller) => {
        controller.startLoad(startPosition);
      });

0: LevelController {hls: Hls, timer: -1, canLoad: true, retryCount: 0, log: ƒ, …}
1: StreamController {_tickTimer: null, _tickInterval: 4878, _tickCallCount: 0, hls: Hls, _boundTick: ƒ, …}
2: AudioTrackController {hls: Hls, timer: -1, canLoad: false, retryCount: 0, log: ƒ, …}
3: AudioStreamController {_tickTimer: null, _tickInterval: null, _tickCallCount: 0, hls: Hls, _boundTick: ƒ, …}
4: SubtitleTrackController {hls: Hls, timer: -1, canLoad: false, retryCount: 0, log: ƒ, …}
5: SubtitleStreamController



StreamController
  startLoad
    this.level = hls.nextLoadLevel
      nextLoadLevel
-->
LevelController
  set level
    this.hls.trigger(Events.LEVEL_SWITCHING
    loadPlaylist
      this.hls.trigger(Events.LEVEL_LOADING
```

LEVEL_LOADING
=== 

```
[playlist-loader.ts]
onLevelLoading  
  this.load  type: PlaylistContextType.LEVEL
    loader.load
---XHR---
loadsuccess
  handleTrackOrLevelPlaylist
    handlePlaylistLoaded
      this.hls.trigger(Events.LEVEL_LOADED
```

LEVEL_LOADED  子m3u8 下载完
====  
```
onLevelLoaded
  set level()
    this.loadPlaylist ----> 子m3u8
      this.hls.trigger(Events.LEVEL_LOADING
```

```
 ???
 
 ---子带参m3u8读取完毕
 
 handlePlaylistLoaded
   case PlaylistContextType.LEVEL
     this.hls.trigger(Events.LEVEL_LOADED
 
 onLevelLoaded
   this.tick();

???

   doTick
     doTickIdle ----- 选frag逻辑在这里
       this.loadFragment --- load init
         super.loadFragment  -- base stream controller
           _loadFragForPlayback
             _doFragLoad
               this.hls.trigger(Events.FRAG_LOADED
??

loadFragment
  _loadInitSegment
    hls.trigger(Events.FRAG_BUFFERED
      
// 处理完init，进行处理下一个m4s
onFragBuffered
  fragBufferedComplete
    this.tick()  
      this.doTick() 
        this.doTickIdle()
          let frag = this.getNextFragment  ！！！！！选frag！
            getInitialLiveFragment
          this.loadFragment
            super.loadFragment
              this._loadFragForPlayback
                this._doFragLoad   //          Loading part url
                ??
                  _loadFragForPlayback
                    this._doFragLoad 
                  _handleFragmentLoadComplete
                    transmuxer.flush(chunkMeta)
                        worker.postMessage({
                          cmd: 'flush',
                          chunkMeta,
                    })
                    
TransmuxerWorker
  self.transmuxer.flush
    flushRemux---> passthough->remux
                    
```

渲染mse


```
onWorkerMessage
  handleTransmuxComplete
    this.onTransmuxComplete
      _handleTransmuxComplete
        bufferFragmentData
          this.hls.trigger(Events.BUFFER_APPENDING, segment)

protected onBufferAppending(
    event: Events.BUFFER_APPENDING,
    eventData: BufferAppendingData
  )
  operationQueue.append
    this.executeNext
      operation.execute();
         this.appendExecutor
           sb.appendBuffer(data);

```

Flush



```
onFragParsed
  blockBuffers
```

```
onFragBuffered
  fragBufferedComplete
```


```
[log] > [level-controller]: live playlist 2 REFRESHED 25-1
level-controller.ts:524 [log] > [level-controller]: Attempt loading level index 2 at sn 26 part 0 with URL-id 0 https://stream.visionular.com/MWQzZWM0MmNmY2ZlZDNlNjMzYmZhZmVkMWM5ODYxNTI/2M/playlist.m3u8?expired=0&_HLS_msn=26&_HLS_part=0
stream-controller.ts:603 [log] > [stream-controller]: Level 2 loaded [11,25], cc [0, 0] duration:30
base-stream-controller.ts:554 [log] > [stream-controller]: Loading part url: playlistsegment0025.1652872407.m4s sn: 25 p: 1 cc: 0 of playlist [11-25] parts [0-5-5] level: 2, target: 50.967
base-stream-controller.ts:1381 [log] > [stream-controller]: IDLE->FRAG_LOADING
buffer-controller.ts:647 [log] > [buffer-controller]: Updating Media Source duration to 52.000
5a9265d0-1df3-4f79-9012-76478582deaa:938 [log] > [transmuxer.ts]: Flushed fragment 25 p: 1 of level 2
base-stream-controller.ts:1381 [log] > [stream-controller]: FRAG_LOADING->PARSING
buffer-controller.ts:854 [info] > hihihi, appending.........
base-stream-controller.ts:1381 [log] > [stream-controller]: PARSING->PARSED
base-stream-controller.ts:499 [log] > [stream-controller]: Buffered main sn: 25 part: 1 of level 2 [22.000,52.000]
base-stream-controller.ts:1381 [log] > [stream-controller]: PARSED->IDLE
base-playlist-controller.ts:120 
```

event
------

- LEVEL_UPDATED
- FRAG_LOADED
- BUFFER_APPENDING
- LEVEL_PTS_UPDATED
- BUFFER_APPENDED
- FRAG_BUFFERED

???
=====

```
tick
  doTick
    doTickIdle
      this.getNextFragment
      loadFragment
      super.loadFragment = _loadFragForPlayback
    onTickEnd
      checkFragmentChanged
        this.hls.trigger(Events.FRAG_CHANGED
```

FRAG_CHANGED
===== 





```
liveSyncPosition
  estimateLiveEdge
```







???
=====

```
handlePlaylistLoaded
this.hls.trigger(Events.LEVEL_LOADED
```


LEVEL_LOADED
=====

```
alignPlaylists
this.hls.trigger(Events.LEVEL_UPDATED
```


BUFFER_APPENDING
======

```
onBufferAppending
  onComplete
  this.hls.trigger(Events.BUFFER_APPENDED
```

BUFFER_APPENDED
=====

```
onBufferAppended
  frag.appendedPTS = 
  
````

？？
====

```
_loadFragForPlayback
_handleFragmentLoadProgress
_handleTransmuxerFlush  
updateLevelTiming
  this.hls.trigger(Events.LEVEL_PTS_UPDATED
```  
  
 
?
====

updateMediaElementDuration







# ivs

- 主m3u8 1.7s 去除ssl 1s 6.2k  https://b0d63d132328.us-east-1.playback.live-video.net   --- 没走加速
- 子m3u8 1.1s 去除ssl 400ms 13k  video-weaver.tyo03.hls.live-video.net
- 2s的ts片（直接首帧）500ms 350k  https://video-edge-785cca.tyo03.hls.live-video.net
- http 1.1

# aurora

- 主m3u8 1.7s 去除ssl 1s 6.2k  https://b0d63d132328.us-east-1.playback.live-video.net   --- 没走加速
- 子m3u8 1.1s 去除ssl 400ms 13k  video-weaver.tyo03.hls.live-video.net
- 2s的ts片（直接首帧）500ms 350k  https://video-edge-785cca.tyo03.hls.live-video.net
- http 1.1

