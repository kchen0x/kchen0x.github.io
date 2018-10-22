title: Linux 服务器/macOS 环境初始化
date: 2018/4/2 20:49:00
tag:
	- Linux
	- Mac
	- Server

---

本手册适用于新创建的 Linux 服务器基本环境初始化，包括连接性能优化、Shell 脚本语言和 Vim 环境优化配置。

其中魔改 BBR 部分视情况开启，如果服务器需要用作高性能网络代理，可以开启提高性能，否则不必开启。**BBR 开启只能在安装其他软件之前进行，若之后开启则会令安装的软件失效。**

如只需要对服务器环境进行配置，不涉及到网络代理；以及本地环境的初始化配置。可直接跳转到 [¶环境配置：切换 Shell](#%E5%88%87%E6%8D%A2+Shell) 部分开始。

## 开启 BBR POWERED（可选）

[参考并使用了 xratzh 的魔改 BBR](https://github.com/xratzh/CBBR)。

### Debian/Ubuntu（64位）开启魔改 BBR

- 查看操作系统版本：

```
head -n 1 /etc/issue
```

- Ubuntu14.04 需要提前：

```
sudo apt-get install -y software-properties-common
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get -y install g++-4.9
```

- Debian 9 需要提前：

```
wget http://snapshot.debian.org/archive/debian/\
20150123T220434Z/pool/main/o/openssl/libssl1.0.0_1.0.2-1_amd64.deb
dpkg -i libssl1.0.0_1.0.2-1_amd64.deb
```

**第一步：**

```
apt-get install -y wget && wget --no-check-certificate -O \
D1.sh https://raw.githubusercontent.com/xratzh/CBBR/master/D1.sh \
&& bash D1.sh
```

之后输入 Y 就会重启

**第二步：**

```
wget --no-check-certificate -O D2.sh \
https://raw.githubusercontent.com/xratzh/CBBR/master/D2.sh \
&& bash D2.sh
```

### CentOS 7 开启魔改 BBR

**第一步：**

```
yum install -y wget && wget --no-check-certificate -O C71.sh \
https://raw.githubusercontent.com/\
xratzh/CBBR/master/C71.sh && bash C71.sh
```

之后输入 Y 就会重启

**第二步：**

```
wget --no-check-certificate -O C72.sh \
https://raw.githubusercontent.com/\
xratzh/CBBR/master/C72.sh && bash C72.sh
```

### CentOS 6 开启魔改 BBR

**第一步：**

```
yum install -y wget && wget --no-check-certificate -O \
C61.sh https://raw.githubusercontent.com/xratzh/\
CBBR/master/C61.sh && bash C61.sh
```

之后输入 Y 就会重启

**第二步：**

```
wget --no-check-certificate -O C62.sh \
https://raw.githubusercontent.com/xratzh/\
CBBR/master/C62.sh && bash C62.sh
```

### Tips

- **存在对于其他内核的删除，只保留 4.11.8 内核**
- 为什么不能一键，因为 Linux 内核在 4.0 后支持不重启更换，但是 CentOS 和 Debian/Ubuntu 很多的内核都是3.X 的版本，Debian 9 和 Ubuntu16.04 则是 4.X 的版本。
- Xratzh 压力测试，发现 BBR 和魔改 BBR 都开启时会达到最快（个人验证）
- 脚本里加入了对内核的锁定，之后 update 时不会变动内核。内核统一选择 4.11.8 版本。
- 由于这个我找到的 CentOS 历史内核的镜像站的网速时快时慢，所以自己下载了上传到 GitHub，这样能保持一个较为稳定的速度。原来的内核地址仍然在脚本里面，只是被添加注释了，如果你不信任我上传的内核，可以自己取消注释使用镜像站的内核下载方式。   
- 部分内容借鉴了[ Vicer 大佬的脚本](https://moeclub.org/2017/06/24/278/)。

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
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/\
oh-my-zsh/master/tools/install.sh -O -)"
```

### 安装 k-zsh

k-zsh 是我配置的一套 Oh My Zsh 设置，包含了常用的插件和更清晰的主题效果。

```
mv ~/.zshrc ~/.zshrc_bac
curl https://raw.githubusercontent.com/kchen0x/\
k-zsh/master/zshrc > ~/.zshrc
``` 

k-zsh 中配置了部分插件需要单独安装一下才能生效：

```
git clone https://github.com/zsh-users/zsh-autosuggestions \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

其中常用的插件是 `z`，快速跳转到相应目录。还有 `x`，快速解压缩文件等。

## 安装 k-vim 服务器简化版

k-vim for server 是 wklken 配置的 k-vim 的服务器版本，包含了许多简单有用的设置，处于便捷考虑 `perfix` 按键已经被修改为 `,` 键。

```
mv ~/.vimrc ~/.vimrc_bak
curl https://raw.githubusercontent.com/wklken/\
vim-for-server/master/vimrc > ~/.vimrc
```

如果需要安装完整非便携版的 k-vim，请根据 https://github.com/kchen0x/k-vim 进行操作。

## 安装 k-tmux

确保使用 2.3 以上版本的 tmux： `tmux -V`。

```bash
mv ~/.tmux.conf ~/.tmux.conf_bak
curl https://raw.githubusercontent.com/kchen0x/k-tmux/master/tmux.conf \
> ~/.tmux.conf
```

安装插件管理

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

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