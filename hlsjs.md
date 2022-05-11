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
          let frag = this.getNextFragment
            getInitialLiveFragment
          this.loadFragment
            super.loadFragment
              this._loadFragForPlayback
                this._doFragLoad
                  _handleFragmentLoadComplete
                    transmuxer.flush(chunkMeta)
                        worker.postMessage({
                          cmd: 'flush',
                          chunkMeta,
                    })
                    
TransmuxerWorker
  self.transmuxer.flush
    flushRemux
                    
```
