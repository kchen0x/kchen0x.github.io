title: 为你量身定制的终端复用器 k-tmux
date: 2016/11/18 10:03:42
updated: 2018/11/5 00:22:33
tags:
    - tmux
    - terminal
    - multiplexer
---

本文是项目 k-tmux 的说明文档，介绍了一套 tmux 的配置。本配置受到 @wlken 的启发。

ChangeLog: 2018-10-19 更新了弃用的配置命令，完善对鼠标的支持，修改了关闭窗口的键位绑定，安装并使用插件，增加对 prefix 状态和 buffer 复制状态的显示。

<!--more-->

## 什么是 tmux

简单点说，tmux 是个终端复用器，允许你在一个终端中同时开启多个会话。

![display](https://camo.githubusercontent.com/d559d2e46c5484e8c978e75a0c2823c554312d72/687474703a2f2f692e696d6775722e636f6d2f4d6b54415a4a702e706e67)

为什么要使用 tmux 呢？

通俗一点说，通常情况下，每次我们打开一个终端窗口的时候，其实我们是创建了一个和操作系统内核连接的会话。会话可以是本地的，也可以是通过 ssh 连接的。如果是使用 ssh 连接的，一旦连接中断了（网络问题、或者客户端不小心关掉了），那么整个会话也就断了，在会话中执行的命令和运行的程序都会中止。

使用 tmux 就不一样了。tmux 首先会创建一个 server 进程，你可以在 server 进程下创建会话（session），这里的会话有别于普通的会话，它是托管在 server 进程下的，只要 server 进程不被中止，session 不会因为意外中止（例如和 ssh 客户端断开连接）。任何时候你可以通过 `attach` 来接入这个会话。

下面我们看两个简单的场景：

你在公司/实验室开启了一个会话连接到本地或者 ssh 到某个服务器运行一个程序。程序持续运行着，还没有出结果，你得下班/放学回家了，放心的合上电脑走人即可。回到家，ssh 连接上服务器，`attach` 到之前的会话，你现在可以看结果了。

或者：

你连上某个服务器，打开 tmux，开启了一些需要后台长期执行的服务，然后关闭终端，走人。不用担心自己的服务进程因为关闭终端而中止。

## 什么是终端复用器

那为什么说 tmux 是终端复用器？

我们来看看 tmux 的层次结构：

![descpription](http://ww1.sinaimg.cn/mw690/7178f37egw1esoxc7hp5oj20gm0bkjs6.jpg)

一个 tmux server 进程下可以托管多个会话（session），一个会话下面可以开启多个窗口（windows）——窗口就像正常终端中的 tab 标签页一样，一个窗口可以切割成多个窗格（pane）——其实窗格才是正常终端中的会话。

这样，我们就可以看出，tmux 在多个层级上做了复用：

- 我们可以通过不同的会话来分组不同的事务，如果几个人共享一个会话，那么彼此的终端界面是一致的，一方输入，另一方可以实时显示，就像在线聊天室一样。
- 如同现代浏览器一样，还可以通过不同的窗口来实现多标签页。
- 可以通过把一个窗口切割成多个窗格的方式，来有效分配窗口显示资源，提高屏幕占用效率。

下面看看 tmux 复用后的实际效果：

![mutiplexer](http://data.kchen.cc/mac_qrsync/e63751170c3cc32863ada94b1527f581.png-960.jpg)

左下角显示了当前我所在的会话名称 `Test` 以及当前窗口和窗格编号 `1-3`，下方中间部分显示了该会话下的窗口，当前激活的是窗口 `[1:htop]`，右下角是系统时间。这是我自己配置的状态栏。

上方主体部分是当前窗口的窗格分割：

![panes](http://data.kchen.cc/mac_qrsync/2fe242a247e55ffecc33467aede4259a.png-960.jpg)

我在 1 中打开了 vim 编写文本，在 2 中使用 `htop` 查看系统进程，在 3 中准备执行某个命令。三个窗格互不干扰，平行工作，而且优雅整洁。

窗口 `[2:~/blog]` 中我还跑着我的博客服务器，和相关的配置等等，两边泾渭分明。

## tmux 的安装和使用

### 安装 tmux

先通过查看 tmux 的版本号，自己是不是已经装了 tmux：

```bash
tmux -V
```

后续配置是针对 `2.5` 版本进行的，请确认自己的 tmux 的版本是 `2.5+`。

#### MacOS

建议使用 `homebrew` 来安装：

```bash
brew install tmux
```

#### Ubuntu

```bash
sudo apt-get install tmux
```

> 如果 apt 源里面的 tmux 版本过于老旧，可以选择使用下面的办法从源码编译安装最新版本的 tmux。

#### CentOS

这个要自己编译安装最新版本，我们先安装依赖：

```bash
yum install gcc kernel-devel make ncurses-devel
```

下载 `libevent` 并编译安装

```bash
curl -OL https://github.com/libevent/libevent/releases/download/release-2.0.22-stable/libevent-2.0.22-stable.tar.gz
tar -xvzf libevent-2.0.22-stable.tar.gz
cd libevent-2.0.22-stable
./configure --prefix=/usr/local
make
sudo make install
```

下载 tmux v2.8 并编译安装

```bash
curl -OL https://github.com/tmux/tmux/releases/download/2.8/tmux-2.8.tar.gz
tar -xvzf tmux-2.8.tar.gz
cd tmux-2.8
./configure && make
sudo make install
```

### 安装 k-tmux 配置

#### 推荐方式

```bash
1. 如果需要的话，备份原有 tmux 配置

cp ~/.tmux.conf ~/.tmux.conf_bak

2. 获取配置文件

curl https://raw.githubusercontent.com/kchen0x/k-tmux/master/tmux.conf > ~/.tmux.conf

3. 完成安装
```

#### 使用 Github 软链接方式

```bash
git clone https://github.com/kchen0x/k-tmux.git
ln -s $PWD/k-tmux/tmux.conf ~/.tmux.conf
```

> 若使用软链接的方式，不要删除 k-tmux 的真实目录。

#### 插件

k-tmux 的配置中使用了 TPM(Tmux Plugin Manager) 来管理插件，并且开启了其中的 prefix highlight 功能，可以指示当前的 prefix 状态和复制模式。如要启动需要先安装插件：

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

安装完成后进入 tmux，使用[prefix]加大写`I`重载一次插件即可（[prefix]为`ctrl+a`）。

### 使用 tmux

> 以下教程基于我的个人配置讲解，小部分特性和快捷键与官方默认有出入（不同的地方会做出说明），特此声明。另外，为了能够获得最佳体验，请使用 Solarized dark 配色，详情请见 [「八卦阴阳鱼，谈谈 Solarized 配色」](http://kchen.cc/2016/11/16/solarized-colorscheme/)。

### 简要说明

> 操作前缀 `[PREFIX-]` ：tmux 中所有的命令都需要先按下操作前缀 `ctrl+a`。（官方默认为 `ctrl+b`，已更改，主要是为了和 screen 保持一致，同时也更方便按）

#### 会话操作

> 下面有 `$` 命令行提示符的是在原生终端中输入的直接作用于 tmux server 的命令。此外都在 tmux 的会话中键入。注意区分大小写。

```bash
# 创建, tmux new -s <name-of-my-session> 创建一个新的会话
$ tmux new -s basic

# 在tmux中创建一个会话
[PREFIX-:] new -s <name-of-my-session>

# 分离会话 detach
[PREFIX-d]
[detached (from session basic)]
or
$ tmux detach
or
[PREFIX-Ctrl-z]

# 查看已有会话列表(list-session)
$ tmux ls
basic: 1 windows (created Wed Aug  5 14:54:04 2015) [200x49]

# 在tmux中查看会话列表并切换
[PREFIX-s]

# 连接会话(只有一个)
$ tmux attach
$ tmux attach -t basic
$ tmux a -t basic

# 杀掉会话
$ tmux kill-session -t

# 重命名会话
[PREFIX-$] 之后输入名字回车
```

#### 窗口操作

```bash
# 创建一个新的窗口
[PREFIX-c]

# 重命名一个窗口（配置开启动态窗口重命名，会根据当前命令自动更改）
[PREFIX-,] 之后输入名字回车

# 切换到下一个窗口, k-tmux另外配置了PREFIX-t/T
[PREFIX-n]

# 切换到对应窗口
[PREFIX-1/2/3]

# 可视化选择切换到的窗口
[PREFIX-w]

# 查找窗口
[PREFIX-f]

# 退出窗口
exit or
[PREFIX-&] 会有确认
```

#### 窗格操作

```bash
# 垂直/水平分割窗口（原先未修改键位的分割方式是[PREFIX-%]和[PREFIX-"]）
[PREFIX-\] / [PREFIX--]

# 关闭一个窗格, 要确认
[PREFIX-x]

# 或者
exit [窗格里执行]

===begin 窗格切换
# 窗格之间移动，改命令可以在 1500 ms 内连续使用
[PREFIX-hjkl]
or
[PREFIX-↑↓←→]

# 最近使用两个窗口之间切换
[Ctrl-\]

# 展示窗口数字并选择跳转
[PREFIX-q]

# 循环切换
[PREFIX-o]
===end

===begin 窗格大小调整
# 窗格大小调整
[PREFIX-HJKL]

# 暂时把当前窗格最大化，再按一次变回来
[PREFIX-z]
===end

# 关闭当前窗格, 需确认
[PREFIX-x]

# 移动当前窗格到左边/右边
[PREFIX-}] / [PREFIX-{]

# 在新窗口中打开当前窗格
[PREFIX-!]

# 自动轮流切换官方默认的多种布局
[PREFIX-space]
```

#### 复制粘贴

```bash
# 进入复制模式
[PREFIX-[]

# => 可以进行的操作（和 VIM 保持一致）
space/v    开始选择
Ctrl-v     整块选择
hjkl       方向键移动
w/b        向前向后移动一个单词
fx/Fx      行内移动到下一个字符位置
ctrl-b/f   在缓冲区里面翻页
g/G        到缓冲区最顶/底端
/ ?        向下, 向上查找
n/N        查找后下一个, 上一个
Enter/y    复制
[PREFIX-]] 粘贴

# 缓冲区相关
# 复制整个窗格可见区域
[PREFIX-:] capture-pane

# 查看缓冲区内容
[PREFIX-:] show-buffer

# 列出缓冲区列表
[PREFIX-:] list-buffers

# 从缓冲区列表选择并插入到当前窗格
[PREFIX-:] choose-buffer => 回车
```

#### 其他

```bash
# 获得快捷键列表
[PREFIX-?]

# 快速在分割窗格中查看 man 手册 <command>命令
[PREFIX-/ <command>]

# 进入命令模式
[PREFIX-:]

# 一些命令模式下的命令
# 新建窗口
new-window -n console

# 新建并执行命令
new-window -n processes "top"
```

### 鼠标模式

tmux 最大的优点是全键盘操作，使得很多操作变得快捷又优雅。但是缺点也是显而易见的，就是有些时候不大方便：

- 回滚历史记录必须 `PREFIX-[` 进入复制模式，然后通过 VIM 风格进行翻页。（jk和 `ctrl-u`/`ctrl-d`）
- 调整窗格大小不太方便。

所以为了照顾鼠标用户，我在配置中默认开启了鼠标模式（mouse reporting）。

在鼠标模式下，你可以：

- 直接使用滚轮进入复制模式回滚查看历史记录，滚回底部或 `q` 键退出复制模式。
- 选择文本即可完成复制。
- 直接拖动窗格边界调整窗格大小。
- 点击状态栏的标签，切换窗口。

建议大家还是多使用快捷键完成操作，增加工作效率。

想要关闭关闭鼠标模式，请进入命令模式然后：

```bash
# 进入命令模式
[PREFIX-:]

# 关闭鼠标模式
setw -g mouse off
```

当然也可以直接去配置文件里修改相关的配置。想要修改的配置立马生效，可以使用：

```bash
# 重载 tmux 配置
[PREFIX-R]
```

最后再提醒一次大家，善用 `[PREFIX-?]` 快捷键帮助，再配合 `/` 查找命令，使用起来便可如鱼得水。祝大家使用愉快！