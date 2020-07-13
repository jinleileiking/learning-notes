#  vs code提示flutter sdk没安装

重启一下vs就好了。不知道为啥

# vs code版本低，升级不了 

需要升级macos。。。。

# flutter pub get 不行

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

# hover 不能 hotreload

设置了代理


# vscode 找不到device

因为flutter进程没杀干净，退了vscode，把flutter杀干净就好了

# vscode run 不起来了

一直卡在 launching。。。。。我直接用hover run了，--- 


custom pallet 可以开启dart log， 需用代理：

```
[  +14 ms] executing: unzip -t -qq /Users/jinleileiking/lk/flutter/bin/cache/downloads/storage.googleapis.com/flutter_infra/flutter/9a28c3bcf40ce64fee61e807ee3e1395fd6bd954/ios/artifacts.zip
[ +248 ms] Downloading ios tools...
[  +39 ms] Downloading: https://storage.googleapis.com/flutter_infra/flutter/9a28c3bcf40ce64fee61e807ee3e1395fd6bd954/ios/artifacts.zip
```



# plugin 找不到 handlefunc

需要再go里面addplugin。。。。
