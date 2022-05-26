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



```
loadSource
   this.trigger(Events.MANIFEST_LOADING
 
class PlaylistLoader
  onManifestLoading
    this.load = 
  load  


  loadsuccess
    handleMasterPlaylist
      hls.trigger(Events.MANIFEST_LOADED
      
      
      
class LevelController
  onManifestLoaded ------> 主m3u8
    this.hls.trigger(Events.MANIFEST_PARSED
    
    
???

loadsuccess
  handleTrackOrLevelPlaylist
    handlePlaylistLoaded
      this.hls.trigger(Events.LEVEL_LOADED
  
  
onLevelLoaded
  set level()
    this.loadPlaylist ----> 子m3u8
      this.hls.trigger(Events.LEVEL_LOADING
      
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


```


```

