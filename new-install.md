# dotfiles

  * ssh-keygen
  * put public key `id_rsa.pub` to github
  * git init .
  * git remote add origin git@github.com:jinleileiking/DotFiles.git 
  * git checkout master

# vm ubuntu

* sudo apt install open-vm-tools open-vm-tools-desktop ripgrep gcc git tig net-tools zsh jq  curl ssh
* install neovim from source https://github.com/neovim/neovim/
* zplug https://github.com/zplug/zplug
* https://github.com/starship/starship p10k 替代
* fzf git install
* zoxide

## ubuntu

隐藏上面的时间栏

* gnome-shell --version
* gnome-extensions install hidetopbar@mathieu.bidon.ca


## nvim

* 需要源码安装 releases/tag
* https://github.com/junegunn/vim-plug 
* :PlugInstall
* :GoInstallBinaries
* :TSUpdate go (treesitter)
* https://github.com/ryanoasis/vim-devicons/issues/226#issuecomment-492783382
* https://github.com/nvim-treesitter/nvim-treesitter




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





## mac

* brew install global (ctags?gtags?)
* brew install yamllint

## trash 
* pip3 install --user pynvim.  # for deoplete
* :UpdateRemotePlugins # for deoplete


# go

* string to struct : go get github.com/ChimeraCoder/gojson/gojson
