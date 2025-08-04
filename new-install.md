# vm ubuntu

sudo apt install open-vm-tools ripgrep gcc git tig net-tools zsh fzf jq neovim


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

* sudo apt install -y zsh tig
  * ssh-keygen, put keys to github
  * git init .
  * git remote add -t \* -f origin 
  * git checkout master
* https://ohmyz.sh/
* https://github.com/zplug/zplug
* https://github.com/romkatv/powerlevel10k#oh-my-zsh
* [fzf](https://github.com/junegunn/fzf#using-git) ( debian use git install)
* https://github.com/ahmetb/kubectx#installation
* AWS
  * https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
  * aws configure
* /usr/sbin/visudo NOPASSWORD
  * jinlei  ALL=(ALL:ALL) ALL
  * %sudo   ALL=(ALL:ALL) NOPASSWD:ALL
*   https://dandavison.github.io/delta/installation.html
* https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
* https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
* https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html
* https://github.com/ahmetb/kubectx#manual-installation-macos-and-linux
* https://github.com/BurntSushi/ripgrep


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



# nvim

* debian [需要源码安装](https://github.com/neovim/neovim/releases/tag/v0.7.2)  
*  sudo apt install  ./nvim-linux64.deb
* 安装：https://github.com/junegunn/vim-plug 
* :PlugInstall
* :GoInstallBinaries
* brew install global (ctags?gtags?)
* brew install yamllint
* pip3 install --user pynvim.  # for deoplete
* :TSUpdate go (treesitter)
* :UpdateRemotePlugins # for deoplete
* https://github.com/ryanoasis/vim-devicons/issues/226#issuecomment-492783382
* https://github.com/nvim-treesitter/nvim-treesitter


# go

* string to struct : go get github.com/ChimeraCoder/gojson/gojson
