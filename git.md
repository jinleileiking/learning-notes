* 提高gitclone 速度

 git config --global   url."https://github.com.cnpmjs.org".insteadOf "https://github.com"
 
 然后用https地址clone
 
 取消：
 
 git config --global --unset url."https://github.com.cnpmjs.org".insteadOf "https://github.com"



* merge rebase

https://www.cnblogs.com/michael-xiang/p/13179837.html



```
$ git describe --tags
tag1-2-g026498b
```

```
2:表示自打tag tag1 以来有2次提交(commit)g026498b：g 为git的缩写
```
