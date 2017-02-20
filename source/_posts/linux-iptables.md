title: Linux - iptable 的使用
date: 2017/02/20 12:39:36
updated: 
tags:
    - Linux
---

`ipatable` 是 Linux 内核自带的防火墙解决方案。它通过对经由网络的每一个包进行一系列的规则验证来决定该如何处理。

我们可以匹配协议类型、源地址和目的地址或者端口、接口以及它们和之前的包的关系等等。这些规则会被进一步的结构化成为规则链[^1]。它们可以被按需生成，但是有3种默认的规则：

- `INPUT`[^2]
- `OUTPUT`[^3]
- `FORWARD`[^4]

<!--more-->

列出当前的规则：

```
$ iptables -L -n
```

列出当前的 `NAT`[^5] 规则：

```
$ iptables -L -n -t nat
```

阻断指定 IP 的流量：

```
$ iptables -I INPUT -s 10.10.10.10 -j DROP
```

允许指定 IP 的流量：

```
$ iptables -I INPUT -s 10.10.10.10 -j ACCEPT
```

允许指定端口的流量：

```
$ iptables -I INPUT -p tcp -m tcp --dport 31415 -j ACCEPT
```

`-I` 和 `-A` 参数的区别在于 `-I` 将会把指定的规则插入到规则链的最开始来抑制 `-A` 所增加的附带规则。

我们可以通过以下命令来检查当前路由表的防火墙状态：

```
$ sudo iptables -L -n -v
```

其中 `-L` 列出规则，`-v` 显示详细信息，我们通过 `-n` 标识禁止 DNS 方案来提高列出规则的速度。我们可以看到类似以下的输出：

```
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
  10   15    DROP       all --   *      *    192.64.174.69           0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

其中有一条激活的 `INPUT` 规则，其阻止了来自 192.64.174.69 的所有流量。

[^1]: 网络包将会顺序地被规则链中的规则一一检查。
[^2]: `INPUT` 规则链会处理流入服务器的流量。
[^3]: `OUTPUT` 规则链会处理流出服务的流量。
[^4]: `FORWARD` 规则链会处理流经服务器的流量而不停止它们。
[^5]: `NAT` 规则允许我们重写流量的源地址。一般来说，他们被 Untangle 服务器用来改变请求发起端的 IP 地址，并在流量到达时将信息返回去。