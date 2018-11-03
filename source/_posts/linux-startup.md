title: Linux 服务器/macOS 环境初始化
date: 2018/4/2 20:49:00
tag:
	- Linux
	- Mac
	- Server

---

本手册适用于新创建的 Linux 服务器基本环境初始化，包括连接性能优化、Shell 脚本语言和 Vim 环境优化配置。

其中魔改 BBR 部分视情况开启，如果服务器需要用作高性能网络代理，可以开启提高性能，否则不必开启。**BBR 开启只能在安装其他软件之前进行，若之后开启则会令安装的软件失效。**

如只需要对服务器环境进行配置，不涉及到网络代理；以及本地环境的初始化配置。可直接跳转到 [¶环境配置：切换 Shell](#%E5%88%87%E6%8D%A2-Shell) 部分开始。

<!--more-->

## 开启 BBR POWERED（可选）

[参考并使用了 xratzh 的魔改 BBR](https://github.com/xratzh/CBBR)。

## 安装 Mosh

用来提高 SSH 连接远程服务器的输入延时和移动设备连接的稳定性（除非手动操作，基本不会断开连接），安装完成后，直接使用 `mosh` 代替 `ssh` 即可。

### Debian/Ubuntu 安装 mosh

```
sudo add-apt-repository ppa:keithw/mosh-dev
sudo apt-get update
sudo apt-get install mosh
```

### CentOS7 安装 mosh

```
yum -y install epel-release
yum makecache
yum -y install mosh
```

### Tips

如果在内网无法直接访问yum仓库，可以下面的目录查找mosh及依赖的rpm包

http://dl.fedoraproject.org/pub/epel/7/x86_64/

rpm包是安装首字母分类到各个目录的，例如mosh的rpm包在

http://dl.fedoraproject.org/pub/epel/7/x86_64/m/

安装完成后，可能需要打开防火墙的 60000-61000 UDP 端口供 mosh 使用：

Firewall-cmd：

```
firewall-cmd --zone=public --add-port=60000-61000/udp --permanent
firewall-cmd --reload
```

iptables：

```
sudo iptables -I INPUT 1 -p udp --dport 60000:61000 -j ACCEPT
sudo service iptables save
```

如果出现语言问题，添加到 `bashrc` 或者 `zshrc`：

```
export LC_ALL="en_US.UTF-8"
```

或者更妥善的解决方法是将本地（Mac）的 Shell 环境改为英文：

```
export LC_ALL=en_US.UTF-8  
export LANG=en_US.UTF-8
```

## 切换 Shell

把默认的 bash 切换为 zsh，使用 oh-my-zsh 框架，并使用 k-zsh 配置。

### 什么是 Zsh 和它的框架 Oh-My-Zsh?

Oh-My-Zsh is a framework for [Zsh](http://www.zsh.org/), the Z shell.

* 为了使用 Oh-My-Zsh 框架，必须先安装 Zsh。
    * 运行 `zsh --version` 来确认。
    * 如果结果是 `zsh 4.3.9` 或更高的版本则可以。
* 然后，需要把 Zsh 更改为默认的 shell 语言：
    * 在新的 Terminal 中运行 `echo $SHELL` 来确认。
    * 如果结果是 `/bin/zsh` 或其他 zsh 路径均可。

### 安装并设置 Zsh 为默认 shell 语言

如果需要的话，依照以下步骤来安装 Zsh：

1. 安装 Zsh 有两种主流的办法
    * 直接利用包管理工具， *e.g.* Debian/Ubuntu 下的 `sudo apt-get install zsh` 或者 CentOS 下的 `sudo yum update && sudo yum -y install zsh`。
    * 从 [source 源](http://zsh.sourceforge.net/Arc/source.html)安装， 依照 [instructions from the Zsh FAQ](http://zsh.sourceforge.net/FAQ/zshfaq01.html#l7) 进行即可。
2. 通过运行 `zsh --version` 来验证安装。 如果结果是 `zsh 4.3.9` 或更高的版本则可以。
3. 更改为默认 shell： `chsh -s $(which zsh)`
    * 注意，如果 Zsh 不在你的授权 shells 列表里面 (`/etc/shells`) 或者你没有权限使用 `chsh`，上述命令不会成功。 如果是这种情况的话， [你可以依照这个方案执行](https://www.google.com/search?q=zsh+default+without+chsh)。
        * Create `.bash_profile` in your home directory and add these lines: 
            * `export SHELL=/bin/zsh`
            * `exec /bin/zsh -l`
        * Update: `.profile` may work as a general solution when default shell is not bash. I'm not sure if `.profile` may be called by Zsh as well that it could go redundant but we can do it safely with a simple check:
            * `export SHELL=/bin/zsh`
            * `[ -z "$ZSH_VERSION" ] && exec /bin/zsh -l`
4. 登出并重新登入来使用新的默认 shell。
5. 通过运行 `echo $SHELL` 来确认， 如果结果是 `/bin/zsh` 或其他 zsh 路径则安装并切换成功。

### Oh My Zsh 安装

你可以在终端中通过以下任意一条命令来安装 Oh My Zsh。你可以使用 `curl` 或者 `wget`。

#### 通过 curl

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/\
oh-my-zsh/master/tools/install.sh)"
```

#### 通过 wget

```shell
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

### 安装 k-zsh

k-zsh 是我配置的一套 Oh My Zsh 设置，包含了常用的插件和更清晰的主题效果。

```
mv ~/.zshrc ~/.zshrc_bac
curl https://raw.githubusercontent.com/kchen0x/k-zsh/master/zshrc > ~/.zshrc
``` 

k-zsh 中配置了部分插件需要单独安装一下才能生效：

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

其中常用的插件是 `z`，快速跳转到相应目录。还有 `x`，快速解压缩文件等。

## 安装 k-vim 服务器简化版

k-vim for server 是 wklken 配置的 k-vim 的服务器版本，包含了许多简单有用的设置，处于便捷考虑 `perfix` 按键已经被修改为 `,` 键。

```
mv ~/.vimrc ~/.vimrc_bak
curl https://raw.githubusercontent.com/wklken/vim-for-server/master/vimrc > ~/.vimrc
```

如果需要安装完整非便携版的 k-vim，请根据 https://github.com/kchen0x/k-vim 进行操作。

## 安装 k-tmux

确保使用 2.3 以上版本的 tmux： `tmux -V`。

```bash
mv ~/.tmux.conf ~/.tmux.conf_bak
curl https://raw.githubusercontent.com/kchen0x/k-tmux/master/tmux.conf > ~/.tmux.conf
```

安装插件管理

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

安装完成后进入 tmux，使用[prefix]加大写`I`重载一次插件即可（[prefix]为`ctrl+a`）。

## 集群监控 ServerStatus（中文版）

### 自动部署

**【服务端】**

```
wget https://raw.githubusercontent.com/cppla/ServerStatus/master/autodeploy/config.json
docker run -d --restart=always --name=serverstatus -v {$path}/config.json:/ServerStatus/server/config.json -p {$port}:80 -p {$port}:35601 cppla/serverstatus

eg:
docker run -d --restart=always --name=serverstatus -v ~/config.json:/ServerStatus/server/config.json -p 80:80 -p 35601:35601 cppla/serverstatus
```

**【客户端】**

```
wget --no-check-certificate -qO client-linux.py 'https://raw.githubusercontent.com/cppla/ServerStatus/master/clients/client-linux.py' && nohup python client-linux.py SERVER={$SERVER} USER={$USER} PASSWORD={$PASSWORD} >/dev/null 2>&1 &

eg:
wget --no-check-certificate -qO client-linux.py 'https://raw.githubusercontent.com/cppla/ServerStatus/master/clients/client-linux.py' && nohup python client-linux.py SERVER=45.79.67.132 USER=s04  >/dev/null 2>&1 &
```

### 手动部署

查看[官方 Github](https://github.com/BotoX/ServerStatus)

## 安装网络监控 BitMeter

在 [官网下载](https://codebox.net/pages/bitmeteros/downloads) 预编译的二进制包。

安装：

```bash
sudo dpkg -i <target.deb>
```

启动监控服务：

```bash
sudo bmdb webremote
sudo bmdb webstop
sudo bmdb webstart
```

然后默认可以在 `localhost:2605/index.html` 访问到数据。

BitMeterOS 提供了3个工具可以使用：

```bash
bmclient -h
bmdb help
bmsync -h
```

### NetHogs 软件

同时，可以安装 NetHogs 来观察每个进程的实时网速：

```bash
sudo apt install nethogs

sudo nethogs
```