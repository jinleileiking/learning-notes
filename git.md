# 提高gitclone 速度

- git config --global   url."https://github.com.cnpmjs.org".insteadOf "https://github.com"
- 然后用https地址clone
- 取消：
- git config --global --unset url."https://github.com.cnpmjs.org".insteadOf "https://github.com"



# merge rebase

https://www.cnblogs.com/michael-xiang/p/13179837.html

# 获得tag

```
$ git describe --tags
tag1-2-g026498b
2:表示自打tag tag1 以来有2次提交(commit)g026498b：g 为git的缩写
```

# 其他

* git commit --amend不好使： 如果vim有问题，会出出现这种情况，用 core editor = nvim
* 获取最新tag: `git -c 'versionsort.suffix=-' \
    ls-remote --exit-code --refs --sort='version:refname' --tags <repository> '*.*.*'`
* 删tag：
```
//Delete remote:
git push -d origin $(git tag -l "tag_prefix*")

// Delete local:
git tag -d $(git tag -l "tag_prefix*")
```
