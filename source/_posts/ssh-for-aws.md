title: 启用密码登录 AWS
date:  2016/11/14 16:32:24
updated: 
tags:
    - AWS
---

> 网上教程参差不齐，索性自己研究完了写个教程。

众所周知，AWS 使用 PEM 密钥验证登录，这样做是 Amazon 出于相对安全考虑做的设置。但是，在安全要求并不算太高的情况下，使用密码登录会更加的方便（因为有很多时候我们并不好随身带着密钥走）。本文的目的就是配置 AWS 可以使用用户名和密码登录，或开启 root 登录权限。

<!--more-->

既然要使用密码登录，第一步就是要设置密码。先给根用户设置密码：

```bash
sudo passwd root
```

然后切换到根用户：

```bash
su
```

再给 `ec2-user` 设置密码：

```bash
passwd ec2-user
```

修改 SSH 登录配置：

```bash
vi /etc/ssh/sshd_config
```

修改并解注其中：

```bash
PasswordAuthentication yes
```

以启用使用密码登录。

另外，如果你还希望直接使用 `root` 用户登录的话，修改并解注其中：

```bash
PermitRootLogin yes
```

最后，重启 SSH Service 即可，这里我选择直接重启实例（因为每种实例需要重启的地方都不同，许多教程就在这里坑了）。

现在，你就可以使用密码登录 AWS 了。