title: Linux - 用户与权限
date: 2017/02/21 13:19:39
updated: 
tags:
    - Linux
    - Tips
---

使用 Linux 系统，不免会和**用户**和**权限**打交道，本文介绍了**根权限**和文件的**权限属性**等概念和应用。

<!--more-->

## su 和 sudo

在 Linux 中，`su` 命令和 `sudo` 命令有着十分巨大的区别：

- `su` 命令会把你切换到根用户 `root`
- `sudo` 会使用根权限来执行命令

简单的说来，`sudo` 其实就是代表其他授权的用户来执行 **root 命令**的二进制 **setuid**。

我们可以通过修改（需要 root 权限）下列文件的中的用户列表，来决定哪些用户可以执行 `sudo` 命令：

```bash
$ sudo /usr/sbin/visudo
```

默认情况下，这个列表如下所示：

```bash
#User privilege specification
root ALL=(ALL) ALL
```

每一个 `sudo` 行的语法是：

```bash
user machine=(effective_user) command
```

通过上面的语法，我们可以授予某个用户 **root 权限**，其中每一个域代表：

- `user` 是新的 `sudo` 用户的用户名
- `machine` 是 `sudo` 生效的主机名
- `effective_user` 代表被允许执行命令的有效用户
- `command` 代表这这个用户可以执行的一系列命令

## 详解 umask

每一个文件和文件夹在被创建的时候都会被赋予一定的**权限属性**，这些值可以通过 `umask` 来指定。正如 `umask` 的名称所显示的那样，这个值本身其实就是一个可以**禁用**相应权限属性的**掩码**。

> **掩码**表示一个有效的 4 位 8 进制数值。如果把少于 4 位的数值作为参数传入，高位会被用 `0` 补全。

默认情况下，**文件夹**在被创建的时候能获取的权限属性是 `777` （`rwxrwxrwx`），**文件**在被创建的时候能获取的权限属性是 `666` （`rw-rw-rw-`），二者的值都可以被被 `umask` 的掩码给**减掉**。

**掩码过程**相当于禁止掉相应的**权限位**，如果相应位上本身就不具备权限，那么掩码就不会起作用。比如说，如果使用 值为 `111` 的 `umask` 来创建一个文件，并不会影响文件的最终权限：

```bash
rw-rw-rw-
# masking x bit still yields
rw-rw-rw-
```

但是，如果 `umask` 的值是 333 的话，情况就不一样了，`w` 位和 `x` 位的权限都会被禁止掉：

```bash
rw-rw-rw-
# masking w and x bits
r--r--r--
```

我们可以这样来查看当前的 `umask` 值：

```bash
$ umask
0022
#these would be the permissions
#for a new file
$ touch new-file
$ ls -l new-file
-rw-r--r-- 1 user group 0 new-file

#for a new dir
$ mkdir new-dir
$ ls -l new-dir
drwxr-xr-x 2 user group 4096 ./
```

如果我们要把当前**会话**中的 `umask` 设定为 `077` 的话，可以执行：

```bash
$ umask 077
#or
$ umask u+rwx,g-rwx,o-rwx
#or
$ umask u=rwx,g=,o=

# + enables specified permissions
# - disables specified permissions
# = enables specified,disables the others

$ umask
0077
```

如果想要系统上的所有用户或者指定用户都使用设定的 `umask` 值的话，我们需要把相应的设定写入 `/etc/profile` 或者指定的 `~/.bashrc` 文件中去。

