title: 如何在公司网络中保护上网隐私不被监控
date: 2016/09/20 06:19:23
updated: 
tags:
    - 网络
---

## 优点

选用 Shadowsocks 配合 Proxifier 来制定代理规则，FireFox 来实现代理浏览。

可以有主力浏览器 Chrome、Safari 用于办公的同时，FireFox 实现完全隐私的网页浏览。在公司网络下可以有选择的暴露自己浏览记录，公共网络下可以完全保护自己不被窥探。（如果使用 FireFox 作为主力浏览器的话，同样可以用 Chrome 来作为隐私浏览器）

<!--more-->

## Shadowsock 的设置

首先需要有 Shadowsocks 的服务器，无论是去买还是还是自己搭，服务器的地址选在什么地方（这决定了访问速度和是否可以翻墙）都可以。

然后在 https://shadowsocks.org/en/index.html 下载并安装 Shadowsocks 的客户端，安装完后打开并添加新的代理服务器。

## Proxifier 的设置

### 添加 Proxies 

可以选择Https、Socks4、Socks5，此处我们选择配合 Shadowsocks 使用，所以选择Socks5。

地址填写127.0.0.1，端口1080，协议SOCKS Version 5，完成。

### 添加 Rules ：

首先确定有以下三条 Rules：

- Applications: Shadowsocks, Target: Any, Action: Direct
- Applications: Any, Target: localhost; 127.0.0.1; ::1; %ComputerName%, Action: Direct
- Applications: Any, Target: Any, Action: Direct

然后添加 Firefox 来实现代理：

- Applications: fi*fox, Target: Any, Action: Proxy SOCKS5

> Tips: 其实在这里你可以书写各种各样的规则来决定自己其他网络服务（比如聊天软件）经过代理加密。

### 配置远程 DNS 解析

在 DNS 中勾选 Resolve hostname through proxy 即可在服务器端解析 DNS， 保证本地信息不会泄露。

## 配置 FireFox 隐蔽所有上网信息

至此使用 FireFox 上网产生的所有信息从本地到服务器端已经处于加密状态，无法在这一环节被任何手段监听，但是如果你是用的是 Autoproxy 而不是 Proxifier 做的浏览器代理，就会在本地解析 DNS 这样访问的 DNS 解析就会被监听，所以需要在 FireFox 中作如下改动：

- 打开 FireFox，在地址栏输入 `about:fonfig`
- 搜索 `network.dns.disableIPv6`，双击把值改成 True （如果服务器不支持 ipv6）
- 搜索 `network.proxy.socks_remote_dns`，双击把值改成 True （如果使用 Autoproxy 进行代理）

打开 FireFox 并启动无痕浏览模式，这样使用 FireFox 的所有浏览记录都将不会被监听，可以放心使用了。