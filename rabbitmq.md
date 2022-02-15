* delete queues: https://stackoverflow.com/questions/11459676/delete-all-the-queues-from-rabbitmq `rabbitmqctl list_queues --quiet | grep -v name | awk '{print $1}' | xargs -n 1 rabbitmqctl delete_queue`
* 发送不出去问题排查：磁盘low watermark，channel 为block 改`disk_free_limit.absolute = 50MB`
* 总是要500m内存，导致内存不够： 经排查，代码initcontainer至少要500m，写死了，需要改代码
