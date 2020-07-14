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

尝试把这个artifacts.zip挪到这个目录，后续又报了一堆错，好像vscode不能用设置的代理，需要改zsh之类的。于是我看了一下ps，把命令弄了出来，试试：

`/Users/jinleileiking/lk/flutter/bin/cache/dart-sdk/bin/dart --disable-dart-dev --packages=/Users/jinleileiking/lk/flutter/packages/flutter_tools/.packages /Users/jinleileiking/lk/flutter/bin/cache/flutter_tools.snapshot run --machine --target lib/main.dart -d 23F33AAC-D07A-42D1-93D2-DD35E5A8DB52 --track-widget-creation --dart-define=flutter.inspector.structuredErrors=true --start-paused --web-server-debug-protocol ws --web-allow-expose-url -v`

命令复制的不对，还是不行，最终：

在命令行 ` open  /Applications/_green/Visual\ Studio\ Code.app`


# plugin 找不到 handlefunc

需要再go里面addplugin。。。。


# 编译出错                [!] Invalid `Podfile` file: no implicit conversion of nil into String.

ruby太老 -》 rvm 更新 -》 rvm install ruby-head rvm 整了半天，不好使。用rbenv没装成功，最后不用rvm，直接用系统的ruby2.6就好了。。。。


# flutter devices 找不到 mi note 3 

在手机要打开usb调试。。。。

# android 模拟器跑不起来

提示platform version 28没linsence，在sdk manager安装28 platform 的sdk就行了。


#  Could not resolve all files for configuration ':video_player:androidApis'.


