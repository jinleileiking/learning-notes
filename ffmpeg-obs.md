# FFMPEG

## compile

https://trac.ffmpeg.org/wiki/CompilationGuide/Centos


## 265

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



## FAQ

* copy: https://stackoverflow.com/questions/38379412/what-does-copy-do-in-a-ffmpeg-command-line
* 添加sei, 在sps，pps之后： `ffmpeg -i short.flv -c copy -bsf:v "h264_metadata=sei_user_data='086f3693-b7b3-4f2c-9653-21492feee5b8+hello'" short1.flv`
* 查看sei: `ffmpeg -i ./short.flv -c:v copy -bsf:v trace_headers -f null -`
* ffmpeg 支持sei : https://wangtaot.github.io/2020/11/09/ffmpeg%E6%8F%92%E5%85%A5sei%E5%AE%9E%E8%B7%B5/
* debug:  `lldb --  ./ffmpeg_g -re -stream_loop -1 -i https://1.mp4   -g 50 -keyint_min 50 -sc_threshold 0 -f hls -hls_time 2   test.m3u8`
* ffmpeg_g 是有调试信息的，可以debug 但是会乱`./configure --disable-optimizations  --disable-stripping`
* debug: https://lldb.llvm.org/use/tutorial.html
- `-hls_segment_type fmp4`  m4s
- hls muxer: https://www.jianshu.com/p/98ff1c49f232
- 加`#EXT-X-PROGRAM-DATE-TIME:2022-03-01T18:23:40.021+0800` `-hls_flags program_date_time`
- `ffmpeg -h muxer=hls`


## 整体流程

```
input_thread
  while(1)
  av_read_frame
    read_frame_internal
      while (!got_packet && !s->internal->parse_queue)
      ff_read_packet
        ret = s->iformat->read_packet(s, pkt)        ---- demux

main
  transcode
    init_input_threads
      init_input_thread
        av_thread_message_queue_alloc
        pthread_create(input_thread)
    ...
    while (!received_sigterm)
    transcode_step
      process_input
        get_input_packet
        process_input_packet
          while (ist->decoding_needed) {  ---   一个packet有多个frame在这循环
          decode_video      ---- decode
            decode
            send_frame_to_filters           ---- filter
              ifilter_send_frame
          if (!got_output); break
          
      reap_filters
        for (i = 0; i < nb_output_streams; i++)
        见下
 ```

## write hls

* https://blog.csdn.net/leixiaohua1020/article/details/39760711?spm=1001.2014.3001.5502
* reap_filters
  * init_output_stream_wrapper --> 
    * init_output_stream  -> check_init_output_file -> avformat_write_header
      * avformat_init_output -> init_muxer  --> `if ((ret = s->oformat->init(s)) < 0) {` -> hls_init 
      * `ret = s->oformat->write_header(s);` -> hls_write_header 
  * output_packet --> hls_write_packet 
   
```
hls_write_packet
  hlsenc_io_open
  flush_dynbuf
    av_write_frame
      s->oformat->write_packet = mov_write_packet
        mov_flush_fragment
      flush_if_needed
        avio_flush
          flush_buffer
    avio_write
    avio_flush
      flush_buffer
        writeout
          s->write_packet = ffurl_write
            retry_transfer_wrapper
              transfer_func = file_write
                write ---> 落盘m4s, m3u8
  hlsenc_io_close
    ff_format_io_close
      s->io_close = io_close_default
        avio_close
          avio_flush

```

- 打开 fmp4
```
  * frame #0: 0x0000000100514228 ffmpeg_g`io_open_default(s=0x0000000103809e00, pb=0x000000010392ea30, url="test0.m4s", flags=2, options=0x000000016fdfdce8) at options.c:174:17
    frame #1: 0x0000000100445d90 ffmpeg_g`hlsenc_io_open(s=0x0000000103809e00, pb=0x000000010392ea30, filename="test0.m4s", options=0x000000016fdfdce8) at hlsenc.c:324:15
    frame #2: 0x0000000100443498 ffmpeg_g`hls_write_packet(s=0x0000000103809e00, pkt=0x000000016fdfdee8) at hlsenc.c:2755:23
    frame #3: 0x00000001004e3f34 ffmpeg_g`write_packet(s=0x0000000103809e00, pkt=0x000000016fdfdee8) at mux.c:749:15
    frame #4: 0x00000001004e21f0 ffmpeg_g`interleaved_write_packet(s=0x0000000103809e00, pkt=0x0000000000000000, flush=0) at mux.c:1124:15
    frame #5: 0x00000001004e305c ffmpeg_g`write_packet_common(s=0x0000000103809e00, st=0x0000000103005170, pkt=0x000000010303a7a0, interleaved=1) at mux.c:1151:16
    frame #6: 0x00000001004e20f8 ffmpeg_g`write_packets_common(s=0x0000000103809e00, pkt=0x000000010303a7a0, interleaved=1) at mux.c:1208:16
    frame #7: 0x00000001004e2138 ffmpeg_g`av_interleaved_write_frame(s=0x0000000103809e00, pkt=0x000000010303a7a0) at mux.c:1264:15
    frame #8: 0x000000010002c0b0 ffmpeg_g`write_packet(of=0x0000000103106990, pkt=0x000000010303a7a0, ost=0x0000000103005610, unqueue=0) at ffmpeg.c:895:15
    frame #9: 0x00000001000314a0 ffmpeg_g`output_packet(of=0x0000000103106990, pkt=0x000000010303a7a0, ost=0x0000000103005610, eof=0) at ffmpeg.c:974:9
    frame #10: 0x000000010003074c ffmpeg_g`do_video_out(of=0x0000000103106990, ost=0x0000000103005610, next_picture=0x000000010310b2f0) at ffmpeg.c:1509:13
    frame #11: 0x000000010002e864 ffmpeg_g`reap_filters(flush=0) at ffmpeg.c:1669:17
    frame #12: 0x00000001000266cc ffmpeg_g`transcode_step at ffmpeg.c:4928:12
    frame #13: 0x0000000100024918 ffmpeg_g`transcode at ffmpeg.c:4972:15
    frame #14: 0x0000000100023eb4 ffmpeg_g`main(argc=9, argv=0x000000016fdfeb80) at ffmpeg.c:5177:9
    frame #15: 0x00000001022b90f4 dyld`start + 520
```



- 写 fmp4 `hls_write_packet`
-  mux.c: `ret = s->oformat->write_packet(s, pkt);¬`

```
  * frame #0: 0x00000001004a649c ffmpeg_g`ff_mov_write_packet(s=0x000000010288fe00, pkt=0x000000010300fef0) at movenc.c:5885:12
    frame #1: 0x00000001004bc024 ffmpeg_g`mov_write_single_packet(s=0x000000010288fe00, pkt=0x000000010300fef0) at movenc.c:5970:12
    frame #2: 0x00000001004a7fcc ffmpeg_g`mov_write_packet(s=0x000000010288fe00, pkt=0x000000010300fef0) at movenc.c:6090:16
    frame #3: 0x00000001004e3f34 ffmpeg_g`write_packet(s=0x000000010288fe00, pkt=0x000000010300fef0) at mux.c:749:15
    frame #4: 0x00000001004e3070 ffmpeg_g`write_packet_common(s=0x000000010288fe00, st=0x000000010301fea0, pkt=0x000000010300fef0, interleaved=0) at mux.c:1153:16
    frame #5: 0x00000001004e20f8 ffmpeg_g`write_packets_common(s=0x000000010288fe00, pkt=0x000000010300fef0, interleaved=0) at mux.c:1208:16
    frame #6: 0x00000001004e1fe4 ffmpeg_g`av_write_frame(s=0x000000010288fe00, in=0x000000016fdfdbe0) at mux.c:1251:11
    frame #7: 0x00000001004e2774 ffmpeg_g`ff_write_chained(dst=0x000000010288fe00, dst_stream=0, pkt=0x000000016fdfdee8, src=0x000000010481a000, interleave=0) at mux.c:1342:27
    frame #8: 0x0000000100443aa4 ffmpeg_g`hls_write_packet(s=0x000000010481a000, pkt=0x000000016fdfdee8) at hlsenc.c:2886:15
    frame #9: 0x00000001004e3f34 ffmpeg_g`write_packet(s=0x000000010481a000, pkt=0x000000016fdfdee8) at mux.c:749:15
    frame #10: 0x00000001004e21f0 ffmpeg_g`interleaved_write_packet(s=0x000000010481a000, pkt=0x0000000000000000, flush=0) at mux.c:1124:15
    frame #11: 0x00000001004e305c ffmpeg_g`write_packet_common(s=0x000000010481a000, st=0x0000000103206de0, pkt=0x000000010323cc70, interleaved=1) at mux.c:1151:16
    frame #12: 0x00000001004e20f8 ffmpeg_g`write_packets_common(s=0x000000010481a000, pkt=0x000000010323cc70, interleaved=1) at mux.c:1208:16
    frame #13: 0x00000001004e2138 ffmpeg_g`av_interleaved_write_frame(s=0x000000010481a000, pkt=0x000000010323cc70) at mux.c:1264:15
    frame #14: 0x000000010002c0b0 ffmpeg_g`write_packet(of=0x0000000103207cb0, pkt=0x000000010323cc70, ost=0x00000001032072f0, unqueue=0) at ffmpeg.c:895:15
    frame #15: 0x00000001000314a0 ffmpeg_g`output_packet(of=0x0000000103207cb0, pkt=0x000000010323cc70, ost=0x00000001032072f0, eof=0) at ffmpeg.c:974:9
    frame #16: 0x000000010003074c ffmpeg_g`do_video_out(of=0x0000000103207cb0, ost=0x00000001032072f0, next_picture=0x0000000103244e70) at ffmpeg.c:1509:13
    frame #17: 0x000000010002e864 ffmpeg_g`reap_filters(flush=0) at ffmpeg.c:1669:17
    frame #18: 0x00000001000266cc ffmpeg_g`transcode_step at ffmpeg.c:4928:12
    frame #19: 0x0000000100024918 ffmpeg_g`transcode at ffmpeg.c:4972:15
    frame #20: 0x0000000100023eb4 ffmpeg_g`main(argc=9, argv=0x000000016fdfeb80) at ffmpeg.c:5177:9
    frame #21: 0x00000001022b90f4 dyld`start + 520

```

- 多码率

```
reap_filters
  for (i = 0; i < nb_output_streams; i++)
```

- init

```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x00000001002f9828 ffmpeg_g`io_open_default(s=0x000000010300c200, pb=0x0000000103098420, url="init.mp4", flags=2, options=0x000000016fdf7688) at options.c:174:25 [opt]
    frame #1: 0x000000010026064c ffmpeg_g`hls_init [inlined] hls_mux_init(s=0x000000010300c200, vs=0x0000000103098400) at hlsenc.c:0 [opt]
    frame #2: 0x0000000100260364 ffmpeg_g`hls_init(s=<unavailable>) at hlsenc.c:3099:20 [opt]
    frame #3: 0x00000001002cf508 ffmpeg_g`avformat_init_output [inlined] init_muxer(s=0x000000010300c200, options=0x00000001023046b8) at mux.c:408:20 [opt]
    frame #4: 0x00000001002cef5c ffmpeg_g`avformat_init_output(s=0x000000010300c200, options=<unavailable>) at mux.c:490:16 [opt]
    frame #5: 0x00000001002cf798 ffmpeg_g`avformat_write_header(s=0x000000010300c200, options=<unavailable>) at mux.c:513:20 [opt]
    frame #6: 0x000000010001f294 ffmpeg_g`check_init_output_file(of=0x00000001023046b0, file_index=0) at ffmpeg.c:3150:11 [opt]
    frame #7: 0x000000010001f0e4 ffmpeg_g`init_output_stream_wrapper [inlined] init_output_stream(ost=0x0000000102305150, frame=<unavailable>, error="", error_len=1024) at ffmpeg.c:3824:11 [opt]
    frame #8: 0x000000010001ef08 ffmpeg_g`init_output_stream_wrapper(ost=0x0000000102305150, frame=<unavailable>, fatal=<unavailable>) at ffmpeg.c:1054:11 [opt]
    frame #9: 0x00000001000209c4 ffmpeg_g`do_video_out(of=0x00000001023046b0, ost=0x0000000102305150, next_picture=0x0000000102206920) at ffmpeg.c:1223:5 [opt]
    frame #10: 0x0000000100020368 ffmpeg_g`reap_filters(flush=0) at ffmpeg.c:1669:17 [opt]
    frame #11: 0x000000010001a48c ffmpeg_g`transcode [inlined] transcode_step at ffmpeg.c:4928:12 [opt]
    frame #12: 0x0000000100018be4 ffmpeg_g`transcode at ffmpeg.c:4972:15 [opt]
    frame #13: 0x0000000100016724 ffmpeg_g`main(argc=<unavailable>, argv=<unavailable>) at ffmpeg.c:5177:9 [opt]
    frame #14: 0x0000000101c390f4 dyld`start + 520
```


## input
```
main
  transcode
    init_input_threads
      init_input_thread
        av_thread_message_queue_alloc
        pthread_create(input_thread)
    ...
    while (!received_sigterm)
    transcode_step
      process_input
        get_input_packet
          get_input_packet_mt      
      

input_thread
  av_read_frame
  av_thread_message_queue_send
 
```

## 改变pts

```
process_input_packet
  pkt->pts += av_rescale_q(ifile->ts_offset, AV_TIME_BASE_Q, ist->st->time_base)
  decode_video
```

## ffmpeg cmd

- `-copyts` ts不变，否则demux后就会改
- `-debug_ts` 打印ts信息


## input_thread_queue

不使用input_thread_queue:   `input -> demux -> decode -> encode -> remux -> output`    
使用：  `input | queue | demux -> encode -> remux -> output`, 即input receive回建立一个线程，通过queue的方式供 demux进行处理    
对于直播场景而言，使用input_thread_queue,这个 thread会不停的调用receive收取tcp数据包，由于直播发包是按照设置的码率来发的，所以只要后续处理跟的上，这个队列不会有积压。实际测试在40-0之间波动。如果后续处理慢，则这个queue会积压到满，ffmpeg也会打一个log进行告警。理论上到达配置值，则ffmpeg的后续处理就有问题了。   
不使用input_thread_queue，直播是否正常的可观测指标为input tcp链接的tcp q size，这个指标不容易观察      
wz_live分支已经有这个queue的打印统计，客户如果使用这个参数，可以使用这个指标来提高系统的可观测性:
* 客户自己看这个值，发现queue满了，就异常了，需要处理
* 我们上报这个值到云，我们监测，发现异常了，提醒客户有异常，提醒客户处理

## ffprobe

* 让ffprobe探流一会停： 

```
-read_intervals will do the trick. e.g. %+2 is to read first 2 sec, %+#2 to read 2 frames.
For example: -read_intervals "%+2"
```

## ffplay

* 低延时: -fflags nobuffer : https://stackoverflow.com/questions/16658873/how-to-minimize-the-delay-in-a-live-streaming-with-ffmpeg. ` ffplay http://$SERVICE_IP/live/1.flv -v debug -fflags nobuffer -fflags discardcorrupt -flags low_delay -framedrop -avioflags -strict experimental`



## zmq

* https://ffmpeg.org/ffmpeg-filters.html#toc-zmq_002c-azmq
* `ffmpeg -re -stream_loop -1     -i main.mp4   -i logo.flv -filter_complex "overlay@my=W-w:H-h-56,zmq"   -f flv  rtmp`
*  `echo Parsed_overlay_0 y 100 | zmqsend`
*  `echo my y 100 | zmqsend`


# obs 

* 低延时：http://help.nuoyun.tv/chapter1/obs-LowLatency.html

