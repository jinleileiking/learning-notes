# compile

https://trac.ffmpeg.org/wiki/CompilationGuide/Centos


# 265

https://www.jianshu.com/p/cba823dddbfe


https://github.com/ksvc/FFmpeg

https://github.com/videolan/x265

```
./configure --enable-static --enable-pic \
        --disable-encoders --enable-encoder=aac --enable-encoder=libx264 --enable-gpl --enable-libx264 --enable-encoder=libx265  --enable-libx265 \
        --disable-decoders --enable-decoder=aac --enable-decoder=h264 --enable-decoder=hevc  \
        --disable-demuxers --enable-demuxer=aac  --enable-demuxer=mpegts --enable-demuxer=flv --enable-demuxer=h264 --enable-demuxer=hevc --enable-demuxer=hls  \
        --disable-muxers --enable-muxer=h264  --enable-muxer=flv   --enable-muxer=mp4 --enable-muxer=null \
        --disable-doc --extra-cflags="-fno-stack-check"   --pkg-config-flags="--static"
```


./configure --list-muxers



# ffmpeg

* copy: https://stackoverflow.com/questions/38379412/what-does-copy-do-in-a-ffmpeg-command-line
* 添加sei, 在sps，pps之后： `ffmpeg -i short.flv -c copy -bsf:v "h264_metadata=sei_user_data='086f3693-b7b3-4f2c-9653-21492feee5b8+hello'" short1.flv`
* 查看sei: `ffmpeg -i ./short.flv -c:v copy -bsf:v trace_headers -f null -`
* ffmpeg 支持sei : https://wangtaot.github.io/2020/11/09/ffmpeg%E6%8F%92%E5%85%A5sei%E5%AE%9E%E8%B7%B5/
* debug:  `lldb --  ./ffmpeg_g -re -stream_loop -1 -i https://1.mp4   -g 50 -keyint_min 50 -sc_threshold 0 -f hls -hls_time 2   test.m3u8`
* ffmpeg_g 是有调试信息的，可以debug
* 


## write hls

* https://blog.csdn.net/leixiaohua1020/article/details/39760711?spm=1001.2014.3001.5502


```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x00000001002f9828 ffmpeg_g`io_open_default(s=0x0000000103010e00, pb=0x00000001030b4820, url="test0.ts", flags=2, options=0x000000016fdfc9a8) at options.c:174:25 [opt]
    frame #1: 0x000000010025eea0 ffmpeg_g`hls_write_packet(s=0x0000000103010e00, pkt=0x000000016fdfcb90) at hlsenc.c:2582:23 [opt]
    frame #2: 0x00000001002d1628 ffmpeg_g`write_packet(s=0x0000000103010e00, pkt=0x000000016fdfcb90) at mux.c:749:15 [opt]
    frame #3: 0x00000001002d13b0 ffmpeg_g`write_packet_common at mux.c:1124:15 [opt]
    frame #4: 0x00000001002d1354 ffmpeg_g`write_packet_common(s=0x0000000103010e00, st=<unavailable>, pkt=<unavailable>, interleaved=1) at mux.c:1151:16 [opt]
    frame #5: 0x00000001002d0328 ffmpeg_g`write_packets_common(s=<unavailable>, pkt=<unavailable>, interleaved=<unavailable>) at mux.c:1208:16 [opt] [artificial]
    frame #6: 0x00000001002d037c ffmpeg_g`av_interleaved_write_frame(s=0x0000000103010e00, pkt=0x0000000102205810) at mux.c:1264:15 [opt]
    frame #7: 0x000000010001fdb4 ffmpeg_g`write_packet(of=<unavailable>, pkt=0x0000000102205810, ost=0x0000000102305040, unqueue=<unavailable>) at ffmpeg.c:895:15 [opt]
    frame #8: 0x0000000100021f64 ffmpeg_g`output_packet(of=<unavailable>, pkt=<unavailable>, ost=<unavailable>, eof=<unavailable>) at ffmpeg.c:974:9 [opt] [artificial]
    frame #9: 0x0000000100021874 ffmpeg_g`do_video_out(of=<unavailable>, ost=0x0000000102305040, next_picture=0x000000010233fdc0) at ffmpeg.c:1509:13 [opt]
    frame #10: 0x0000000100020368 ffmpeg_g`reap_filters(flush=0) at ffmpeg.c:1669:17 [opt]
    frame #11: 0x000000010001a48c ffmpeg_g`transcode [inlined] transcode_step at ffmpeg.c:4928:12 [opt]
    frame #12: 0x0000000100018be4 ffmpeg_g`transcode at ffmpeg.c:4972:15 [opt]
    frame #13: 0x0000000100016724 ffmpeg_g`main(argc=<unavailable>, argv=<unavailable>) at ffmpeg.c:5177:9 [opt]
    frame #14: 0x0000000101c390f4 dyld`start + 520
```





# ffprobe

* 让ffprobe探流一会停： 

```
-read_intervals will do the trick. e.g. %+2 is to read first 2 sec, %+#2 to read 2 frames.
For example: -read_intervals "%+2"
```

# ffplay

* 低延时: -fflags nobuffer : https://stackoverflow.com/questions/16658873/how-to-minimize-the-delay-in-a-live-streaming-with-ffmpeg. ` ffplay http://$SERVICE_IP/live/1.flv -v debug -fflags nobuffer -fflags discardcorrupt -flags low_delay -framedrop -avioflags -strict experimental`

# obs 

* 低延时：http://help.nuoyun.tv/chapter1/obs-LowLatency.html
