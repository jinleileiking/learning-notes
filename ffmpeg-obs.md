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
