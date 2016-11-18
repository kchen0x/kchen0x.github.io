title: iTerm2 与 Tmux 的集成
date: 2016/11/17 08:43:58
updated: 
tags:
    - iTerm2
    - tmux
---



iTerm2 已经集成了 tmux，这意味着什么？

通常情况下，当你使用 tmux 的时候，会在一个「物理」窗口（Window）中创建多个虚拟的窗口。你可以通过在 tmux 中使用各种命令来操作它的环境，但这样也会随之带来一些问题：

- 你需要敲下前缀修饰键来进入 tmux 的命令模式（默认情况下是`control + b`，这和 emacs 中的左移光标是冲突的，而且这也会让与 shell 的交互变得更加困难）。
- 你需要不止一次的使用 ssh 来连接到远程服务器（remote host）以获得不止一个的 tmux 会话（session）窗口。
- 你需要学习 tmux 的命令。
- 你需要开启鼠标报告（mouse reporting）来调整分割窗格（pane）的大小，尽管你并不想启用它。
- 当你使用 tmux 的时候，一些终端模拟器内置的功能不能很好工作，比如说：你并不能像在普通的终端窗口中那般快捷的使用回滚查看历史，同时，tmux 的查找功能也完全跟 iTerm2 的不能比拟。

对于大多数的用户而言，在终端中使用多路器（Multiplexer）是十分好用的工作方式，但是他们并不想接受以上的种种缺陷。

<!--more-->

iTerm2 与 tmux 的集成（iTerm2's tmux integration）就解决了这些痛点。

当你执行 `tmux -CC` 命令时一个新的 tmux 会话就会被创建，一个看上去和普通 iTerm2 窗口没有差别的窗口将会被打开。唯一不同的地方就是，当 iTerms2 退出或者是 ssh 会话丢失时，tmux 会保持运行。你可以重新连接上刚刚 ssh 连接的远程主机，然后执行 `tmux -CC attach` 命令，iTerm2 窗口会重新打开并恢复到断开时相同的状态。那么，一些应用场景就不难想象了：

对于那些常常使用 ssh 的小伙伴来说，你可以：

- 回到家中然后恢复公司的工作环境。
- 不必担心系统升级的电脑重启。

而对于所有小伙伴而言，你可以：

- 通过连接同一个 tmux 会话和别的小伙伴协作（collaborate）。
- 保护自己不因 iTerm2 崩溃（iTerm3 会通过会话修复特性来减轻这种状况）而丢失工作环境。

## 用法

你可以一如往常那般使用 tmux，只需要在末尾加上 `-CC` 参数就可以了，实际上，也就是执行以下任意一个命令：

```bash
tmux -CC
tmux -CC attach
```

当你执行 `tmux -CC` 命令的时候，你将会在终端中看到如下的菜单：

```bash
** tmux mode started **

Command Menu
----------------------------
esc    Detach cleanly.
  X    Force-quit tmux mode.
  L    Toggle logging.
  C    Run tmux command.
```

- 如果你按下 `esc` 键，tmux 窗口会关闭，tmux 客户端也会终止。
- 如果你按下 `esc` 键但是任何事情都没有发生，这说明 tmux 的客户端可能崩溃了或者是除了别的状况。这时按下 `X` 键来强制 iTerm2 退出 tmux 模式。如果真是的 tmux 客户端崩溃的话，你也许会需要执行 `stty sane` 命令来恢复你的终端状态。
- 如果你想提交一个 Bug 的话，可以通过按下 `L` 键来重现问题，tmux 协议命令会被打印到屏幕上。
- 如果你想执行菜单中没有的命令，你可以按下 `C` 键来进入 tmux 命令模式，一个可以输入命令的对话框将会弹出，你可以键入类似 `new-window` 这样的命令。

通常情况下， 大多数的动作都不需要通过键入命令来实现，以下的一些 iTerm2 的动作就可以直接作用于 tmux：

- 关闭会话，标签页（tab）或者是窗口：终止 tmux 会话或窗口。
- 分割窗格：通过 `split-window` 分割 tmux 窗口。
- 调整窗格大小：通过 `resize-pane` 命令调整 tmux 窗格大小。
- 调整窗口大小：告诉 tmux 客户端的大小改变了，重调所有窗口的大小。窗口不会大于它连接（attach）的最小的客户端的大小，一个灰色的区域将会出现的窗口的右下方表明实际窗口的大小超出了 tmux 窗口允许的最大大小。这一原则的一个好处就是所有的 tmux 窗口/标签页都包含完全相同的行数和列数。
- 通过菜单栏 Shell->tmux 创建一个新的窗口或者标签页：创建一个新的 tmux 窗口。
- 通过菜单栏 Shell->tmux->Detach 断开（detach）与 tmux 会话的连接：断开与 tmux 会话的连接，所有 tmux 窗口都会被关闭，你可以通过 `tmux -CC attach` 命令重新与之连上。

## 限制

大多数的限制都将会在接下来的版本中得到解决和改进：

- 在早于2.9版本的 iTerm2 中，你只能同时连接上一个 tmux 会话。在2.9和更新的版本中，你可以同时连接多个 tmux 会话。
- `.tmuxrc` 文件未经测试，可能运转不正常。
- 在早于2.9版本的 iTerm2 中，不能最大化窗格，已经在2.9版本中解决了这一问题。

## 创建 tmux

你需要使用1.8或更高版本的 tmux，在 Mac 上安装 tmux 最简便的方法是使用 `homebrew`：

```bash
brew install tmux
```

> 原文链接 [《iTerm2 and tmux Ingeration》](https://gitlab.com/gnachman/iterm2/wikis/TmuxIntegration)
