* delete queues: https://stackoverflow.com/questions/11459676/delete-all-the-queues-from-rabbitmq
* 发送不出去问题排查：磁盘low watermark，channel 为block 改`disk_free_limit.absolute = 50MB`
