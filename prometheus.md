* 安装参考 http://www.mydlq.club/article/112/
* name 需要 在 metric_relabel_configs里：https://github.com/prometheus/prometheus/issues/5200
* 磁盘利用率： `((node_filesystem_avail_bytes{fstype="xfs"} * 100) / node_filesystem_size_bytes{fstype="xfs"})`
* metric是采集proc/net/底下的文件得出 https://github.com/prometheus/node_exporter/blob/98a40bd712eaeb1fd3ee60ca37e4091c503d8e1c/collector/netdev_linux.go#L53
