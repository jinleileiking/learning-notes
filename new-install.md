# mac

* brew 要用vpn翻墙
* brew update 不好使，就brew doctor
* brew install zsh
* zplug
* fzf ( debian use git install)
* brew install tig
* brew install jq # for pjson not in brew
* iterm `ctrl [ b 麻烦，映射为 ctrl b:  `iterm pref -> key binding -> ^b --> escape + B. ( escape = ^[) ]`
*  brew install httpie. # better curl
*  krew https://krew.sigs.k8s.io/docs/user-guide/setup/install/
* brew install --cask maccy # maccy clipboard

# debian

* fzf ( debian use git install)
* /usr/sbin/visudo NOPASSWORD
  * jinlei  ALL=(ALL:ALL) ALL
  * %sudo   ALL=(ALL:ALL) NOPASSWD:ALL
*  sudo apt install -y zsh tig
*  https://github.com/zplug/zplug
*   chsh -s /usr/bin/zsh
* https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management
* https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
* aws configure
* https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html


# tmux

* https://github.com/tmux/tmux
  * debian
  *  sudo apt install libncurses5-dev
  *  sudo apt install bison
  *  sudo apt install flex
* https://github.com/tmux-plugins/tpm   
  * ` git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm`
  * ` tmux source ~/.tmux.conf`
  * `Press prefix + I (capital i, as in Install) to fetch the plugin.`



# vim

* 安装：https://github.com/junegunn/vim-plug 
* :PlugInstall
* :GoInstallBinaries
* brew install global (ctags?gtags?)
* brew install yamllint
* pip3 install --user pynvim.  # for deoplete
* :UpdateRemotePlugins # for deoplete
* https://github.com/ryanoasis/vim-devicons/issues/226#issuecomment-492783382


# go

* string to struct : go get github.com/ChimeraCoder/gojson/gojson
