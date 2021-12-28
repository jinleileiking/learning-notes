# iterm

* ctrl+A:  `The correct binding is ⌘←  "SEND ESC SEQ"      OH for Home and ⌘→  "SEND ESC SEQ"      OF for End (those are uppercase 'o's not zeros). This simulates actually pressing the Home and End keys, and as such will work in bash, vim, etc.`


# zplug 循环加载插件

zplug clear


# grep or

`grep -e pattern1 -e pattern2 filename`



# rg


` rg wz -g '!tags' -g '!*js*' -g '!*.*~'`


# tz

```
ISO标准类型有：


"2019-06-10" (date-only form)


"2019-06-10T14:48:00" (date-time form)


"2019-06-10T14:48:00.000+09:00" (date-time form with milliseconds and time zone)


"2019-06-10T00:00:00.000Z" (specifying UTC timezone via the ISO date specification，Z is the same with +00:00)

 
```


# httpie

`http --verbose POST  'http://xxxxxxxx/video/v1/live-streams'   X-API-ACCOUNT-ID:10000   simulcast_targets:='[{"stream_key":"dest", "url":"rtmp://zzzzzz/live"}]'`


# jq

`jq -r .data.stream_key`
