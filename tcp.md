```
拥塞算法
cat /proc/sys/net/ipv4/tcp_allowed_congestion_control
cat /proc/sys/net/ipv4/tcp_congestion_control
窗口扩展
cat /proc/sys/net/ipv4/tcp_window_scaling
```




* 可变窗口 发送数据的win，代表我这边能收多少的窗口，对方最多给我发多少
* 阻塞避免 — reno tahoe vegas new reno hybla
* 慢启动
* 选择性确认（sack）— 接收方通知发送方，收到非连续数据块
* 前向确认（fack)
* 快速重传
* 快速恢复

重复确认检测
```
 11 12 13 -> 
11 drop
 <- 11 11  响应12，13 — 我在等待11
 11 -> 
```

* Nagle : 小数据包合并  但在广域网上， 这些小分组则会增加拥塞出现的可能。
* delayed ack：延时发送ack，从而合并ack（其他控制报文），减少包数量


```
肯定确认 -> sliding window
等待期间也进行发送，利用带宽
-> 可变窗口

大的mss会导致分片，一分片，如果一个segment丢了，整个报文段都要重传

接收方发送的序号的含义：

这个序号如果为A,则表示告诉发送方：A-1我收到了，你发A吧

累积确认的bug：

100,101,102,103,104

100 丢失， 而101，102，103，104都到了，接收方返回的ack一直是101，导致发送方不知道这4个已经都收到了。

发送方超时重传，如果传了5个，则浪费了。

二义性：
RTT = 接收X的ACK时间 - 发送X时间
如果一个报文重传，则接收方无法判断是第一个还是第二个的到达时间，故RTT不准
二义性带来的问题：rtt算不准。
解决- karn 只用第一次的ack时间 * 重传次数

估计重传时间的意义：重传定时器尽可能接近RTT，只能大不能小

拥塞崩溃 congestion collapse
慢启动 slow start
启动新连接，拥塞恢复？？？
拥塞窗口 = 1，2，4，8 …。。。。。
拥塞避免 9、10，11 。。。 16

加速递减 multiplicative decrease

拥塞窗口界限 congestion window limit

发送方的拥塞窗口在丢包后调整
allowed window (允许发送包） = min（ack window值 ， congestion window）

路由器尾部丢弃导致，当一个链接导致拥塞，会将所有的链接进入慢启动—> 路由器red

timewait的意义：能够辨别出是新连接还是旧连接，防止老的reset reset掉新的连接

push: 立即发送(telnet)

吞吐率：占用带宽

SWS糊涂窗口综合征？ 
接收app每次只处理一个字节，导致发送方每次都发一个字节，退回到了一发一收状态，浪费带宽
接收方：delayed ack  
只有在可用缓冲区达到缓冲区一半 || 最长报文数据量 才发 —- 不推荐
只有窗口能避免此种情况时，才发
发送方：Nagle，如果刚发完，没有ack，则攒包， 如果ack来了，则发 
```
