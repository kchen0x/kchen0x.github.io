title: SSH 端口转发
date: Nov 11, 2016 9:46 AM
updated: 
tags:
    - SSH
    - 网络
---

## 什么是 SSH

> **SSH** 为 Secure Shell 的缩写，由IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。 SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。

简单的说来 SSH 是一种网络协议，用来进行远程登录，在源主机和目标主机之间形成一条加密的数据通道，有时又称为 SSH Tunnel。

但凡对 Linux 系统不陌生的人，都应该都知道 `ssh` 指令，其最基本的用法为：

```bash
ssh [user@]hostname
```

<!--more-->

表示建立本机到 `hostname` 的远程登录，用户名 `user` 为可选项，缺省时默认使用本机当前用户名。想详细了解远程登录、免密远程登录和中间人攻击的同学可以移步[ SSH 原理与运用（一）：端口转发 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)但是，今天我们想谈的并不是用 SSH 来实现远程登录，而是端口转发。

## 什么是端口转发

我们从一个实际的例子出发来看什么是端口转发。

假设我们现在有三台计算机，SourceHost A 想要访问 DestinationHost C，而两者之间并没有网络连接，但是在中间有一个 MiddleHost B 可以访问到两台计算机。我们便可以通过访问 B 从而将端口数据转发到 C 上，完成通信。一个典型的场景便是，
B 下有一个私有的子网，子网内有 C，而在外网的 A 想要访问 C 上的数据，我们在建立虚拟机时常遇到这样的场景。

![net topology](https://www.processon.com/chart_image/5826cc1fe4b00c4fc87e353e.png)

下面我们通过各种端口转发来看看如何突破限制，使得 A 可以访问到 C 的数据。

### 本地转发

本地转发的命令格式如下：

```bash
ssh -L [bind_address:]port:host:hostport [user@]hostname
```

我们先来看一看 Man Page 上对这个参数的解释：

> Specifies that the given port on the local (client) host is to be forwarded to the given host and port on the remote side. This works by allocating a socket to listen to `port` on the local side, optionally bound to the specified `bind_address`. Whenever a connection is made to this port, the connection is forwarded over the secure channel, and a connection is made to `host` port `hostport` from the remote machine. Port forwardings can also be specified in the configuration file. IPv6 addresses can be specified by enclosing the address in square brackets. Only the superuser can forward privileged ports. By default, the local port is bound in accordance with the `GatewayPorts` setting. However, an explicit `bind_address` may be used to bind the connection to a specific address. The `bind_address` of ''localhost'' indicates that the listening port be bound for local use only, while an empty address or '*' indicates that the port should be available from all interfaces.

如何来理解呢？其实我们可以看到，本地转发的指令相对 SSH 的基本指令，多了一个 `-L` 参数：

```bash
ssh [user@]hostname
ssh -L [bind_address:]port:host:hostport [user@]hostname
```

这意味着我们连接到了远程主机之后还需要将本地端口转发到远程主机去，比如现在我在 B 上执行：

```bash
ssh -L MidlleHsot:7001:localhost:9092 DestinationHost
```

那么在 B 和 C 的隧道建立之后，所有发送到 `7001` 端口的请求，都会通过隧道加密传输到 C，然后在 C 上解析为发送到 `localhost:9092` 的请求。所以，如果现在 A 想访问 C 的 9092 端口，只需要访问 B 的 7001 端口就可以了，同时 B 也可以通过访问自己的 7001 端口发送数据到 C。

下面我们来仔细看一下这个流程：

- 计算机 A 上的应用将数据发送到计算机 B 的 7001 端口上
- 计算机 B 的 SSH Client 会将 7001 端口收到的数据加密并转发到 C 的 SSH Server 上
- 计算机 C 的 SSH Server 会解密收到的数据并将之转发到监听的 localhost:389 端口上
- 最后再将从监听端口返回的数据原路返回以完成整个流程

从上面的例子我们可以看出：

- 对于计算机 A 而言，它并不知道 C 的任何信息，它只与 B 进行交互；
- 对于计算机 C 而言，它只是监听了来自本地主机的端口，甚至可以限定应用只能响应本地请求，C 上的 SSH Server 负责将数据隧道传来的数据发送到 9092 端口；
- 对于计算机 B 而言，它通过 SSH Client 与 C 的 SSH Server 建立了加密的数据隧道，并把本地 7001 端口的数据通过隧道转发出去，这就是所谓的**本地转发**。

需要注意的是：

1. 本例中我们选择了 7001 端口作为本地的监听端口，在选择端口号时要注意非管理员帐号是无权绑定 1-1023 端口的，所以一般是选用一个 1024-65535 之间的并且尚未使用的端口号即可。
2. SSH 端口转发是通过 SSH 连接建立起来的，我们必须保持这个 SSH 连接以使端口转发保持生效。一旦关闭了此连接，相应的端口转发也会随之关闭。
3. 我们只能在建立 SSH 连接的同时创建端口转发，而不能给一个已经存在的 SSH 连接增加端口转发。
4. 对于 `bind_address` 我们可以指定为 `localhost`，这样，只有 B 本机的请求会被转发，来自 A 的会被拒绝；还可以指定为 `<MiddleHostIP>`，这样，只有来自 A 的请求会被转发，来自本机的会被拒绝；也可以不指定或者指定为 `'*'`，这样，所有的接口的请求都会转发。
5. 如果参数中的 `host` 是 `localhost` 或者和 `hostname` 保持一致到的话，说明请求响应会被远程主机 C 自发自收，这时候的端口转发走的上图中绿线部分，主要实现的是让外网的 A 主机通过 B 主机访问 C 主机上只有本地权限或内网权限的服务。这也就是所谓的**内网穿透**[^1](内网穿透即 NAT 穿透，网络连接时术语，指外网与内网的计算机节点进行跨网通信。)。

### 远程转发

现在我们试想新的场景，这次假设由于网络或防火墙的原因我们不能用 SSH 直接从 B 连接到 C 服务器，但是反向连接却是被允许的，所以这次我们得选择使用远程转发来打通隧道。

![net-topology-2](https://www.processon.com/chart_image/58265f04e4b0fa6ffbac5c15.png)

远程转发的命令格式如下：

```bash
ssh -R [bind_address:]port:host:hostport [user@]hostname
```

这次我们在 C 上执行：

```bash
ssh -R MidlleHsot:7001:localhost:9092 MiddleHost
```

来建立隧道。可以看出，其实 `-R` 参数后面的转发规则是一致的，同样是将访问 `MidlleHsot:7001` 的请求通过隧道连接到 C，转发到 `localhost:9092` 端口上去。那么为什么这个就是**远程转发**了呢？

我们可以看到上图中 C 已经变成了 ssh Client，而 B 是 ssh Server，所以需要进行端口转发的规则是在「服务器端」B 进行的，相对于「客户端」的 C 而言，就是**远程转发**了。

### 动态转发

上面的本地转发和远程转发都面临着一个问题，我们需要知道转出端的端口号（例子中就是 C 提供服务的 9092端口）。那如果我们想要访问的服务并不需要端口号怎么办？

> 其实大多数的应用例如：在浏览器上浏览网页，使用聊天软件聊天等等都是不需要指定端口号的。

这个时候我们可以启用动态转发，它的命令语法格式如下：

```bash
ssh -D [bind_address:]port [user@]hostname
```

比如在如下的网络拓扑结构中：

![dynamic-farwording](https://www.processon.com/chart_image/582666f1e4b0fa6ffbac6c2b.png)

A 与 B 就通过 ssh 建立了隧道，把 A 上 7001 的访问请求加密传输到 B 上，通过 B 的网络来响应响应请求。其实在这里 SSH 是创建了一个 SOCKS 代理服务，这个方法常常被用来突破防火墙的限制进行代理服务，只要我在计算机 A 的本地把需要翻墙的访问全部代理到 7001 端口上，那么这个访问就可以通过加密隧道到达墙外网络中去了。今天我们是来讲端口转发，关于利用 ssh 隧道进行 SOCKS 代理翻墙的细节我就不在这里赘述了。

## 总结

回顾一下，今天我们介绍了利用 SSH 完成本地端口转发、远程端口转发和动态端口转发。当我们知道确切的应用端口时，我们根据防火墙或者网络状况的限制来选择使用本地端口转发或者远程端口转发；当我们不知道确切的的应用端口时，就使用动态端口转发来启动 SOCKS 代理。希望大家通过这篇文章可以对 SSH 端口转发形成基本的认识。好了，快打开终端上手练习一下吧。