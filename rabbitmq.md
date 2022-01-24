* delete queues: https://stackoverflow.com/questions/11459676/delete-all-the-queues-from-rabbitmq `rabbitmqctl list_queues --quiet | grep -v name | awk '{print $1}' | xargs -n 1 rabbitmqctl delete_queue`
* 发送不出去问题排查：磁盘low watermark，channel 为block 改`disk_free_limit.absolute = 50MB`
