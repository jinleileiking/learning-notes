# ale

* `ALEInfo` 去查debug
* 配置linter启用某些语音的ale检查，有个默认值，不开就不检查


# vim-go

* 重启go引擎：`:GoBuildTags ''` https://github.com/fatih/vim-go/issues/2550


# cmdline

* 编辑stdout : `echo This is example. | vim -`


# nvim 

- lsp 参考这个即可： http://xfyuan.github.io/2021/02/neovim-builtin-lsp-basic-configuration/
- symbols-outline 显示不出来： https://github.com/ryanoasis/vim-devicons/issues/226#issuecomment-492783382
- c跳转
  - brew install clangd
  - brew install bear
  - ffmpeg 目录， `bear -- make`
  - `local servers = { 'pyright', 'rust_analyzer', 'tsserver' }` 这里加上clangd
  - done

# telescope

- ctrl q 可以把搜索结果送到quickfix
