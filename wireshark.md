# http 过滤 host

"http && http.host contains baidu"


# tshark

## 打印httpbody

 tshark -r baidu.pcapng    -T fields -e http.file_data   -Y 'http.response.code == 200'
