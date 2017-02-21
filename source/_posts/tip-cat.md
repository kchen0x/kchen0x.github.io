title: Linux - cat 命令的实用技巧
date: 2017/02/02 10:58:22
updated: 
tags:
    - Linux
    - Tips
---

GNU 文本程序实用技巧系列之——`cat`。`cat` 这个命令是 UNIX 爱好者所热爱的，也是厌恶 UNIX 的人所憎恶的。

> The `cat` utility reads files sequentially, writing them to the standard output.

<!--more-->

## 使用 cat 合并多个文件

有时候，我们常常需要将几个文件处理成一个文件，并将处理的结果保存到一个单独的输出文件中。`cat` （「concatenate」的缩写）命令可以在其输入上接受一个或多个文件并将它们打印到一个单独的文件中。例如， `cat chapter01 chapter02 chapter03 > book` 将三个 `chapterXX` 文件保存在一个单独的 `book` 文件中。

输入文件按照它们在 `cat` 命令后的排列顺序被打印，因此，要调换输出文件中内容的顺序，就必须先调换输入文件的顺序。此外，当需要处理的文件数目过大而无法手工输入这些文件的名称时，可以使用通配符来匹配文件名，例如， `cat chapter* > book` 可以将当前目录下所有以 `chapter` 开头的文件全部合并输出到 `book` 文件中，记住，**文件名将会按升序排列**。这就意味着你会发现 `chapter13` 被发送到输出中时会在 `chapter2` 之前，`chapter02` 之后。有时候，这会引起一些很有意思的问题。

当 `cat` 的输出没有被重定向到一个文件或另一个命令的标准输入时，`cat` 表现出来的行为与多数命令行工具一样，即将其输出发送到控制台。 这意味着你可以使用 `cat` 来显示文件（也就是 `cat` 命令最常见的实用方法）；例如，你可以使用 `cat /etc/passwd` 来显示系统密码文件的内容。为方便起见，你应该用 `less` 或者 `more` 来查看大文件，如 `less /etc/passwd`。（你可以通过输入 `man less` 学习更多关于 `less` 的知识）。

尽管 `cat` 主要用于合并文件，我们还可以将它用于输入的简单自动处理。例如，你可以使用 一个单独的空白行来除去多行空白行（使用 `-s` 选项），这是一个在你将源代码公诸于世前进行代码整理工作的好办法。遗憾的是，`cat` 并没有用于一次清除所有空白行的选项。但这并不是什么大问题，因为你可以方便地使用 `sed` 命令将这些空白行除去：

## 使用 sed 与 cat 除去空白行

```bash
$ cat -s /etc/X11/XF86Config | sed '/^[[:space:]]*$/d'
...
# Multiple FontPath entries are allowed (they are concatenated together)
# By default, Red Hat 6.0 and later now use a font server independent of
# the X server to render fonts.
    FontPath   "/usr/X11R6/lib/X11/fonts/TrueType"
    FontPath   "unix/:7100"
EndSection
...
```

对于读取配置文件和 HTML 页面的源文件，特别是那些由脚本生成的插入了不必要新行的源文件，以及那些包含大型条件结构（其各个项之间已经用空行分开）的源文件来说，空白行紧缩是一个非常实用的技巧。

`cat` 的另外一个重要的功能是它可以对行进行编号。这种功能对于程序文档的编制以及法律和科学文档的编制很方便。打印在左边的行号使得参考文档的某一部分变得容易。这在编程、科学研究、业务报告或甚至是立法工作中都是非常重要的。 对行进行编号功能有两个选项： `-b` 选项（只能对非空白行进行编号）和 `-n` 选项（可以对所有行进行编号）：

## 使用 cat 对行进行编号

```bash
$ cat -b /etc/X11/XF86Config
...
    14  # Multiple FontPath entries are allowed (they are concatenated together)
    15  # By default, Red Hat 6.0 and later now use a font server independent of
    16  # the X server to render fonts.
    17      FontPath   "/usr/X11R6/lib/X11/fonts/TrueType"
    18      FontPath   "unix/:7100"
    19  EndSection
...
$ cat -n /etc/X11/XF86Config
...
    20  # Multiple FontPath entries are allowed (they are concatenated together)
    21  # By default, Red Hat 6.0 and later now use a font server independent of
    22  # the X server to render fonts.
    23  
    24      FontPath   "/usr/X11R6/lib/X11/fonts/TrueType"
    25      FontPath   "unix/:7100"
    26  
    27  EndSection
...
```

`cat` 还可以在你查看包含如*制表符*这样的**非打印字符**的文件时起帮助作用。你可以用以下选项来显示*制表符*：

- `-T` 将*制表符*显示为 `^I`
- `-v` 显示非打印字符，除了*换行符*和*制表符*，它们使用各自效果相当的「控制序列」。 例如，当你处理一个在 Windows 系统中生成的文件时，这个文件将使用 `Control-M`（ `^M` ）来标记行的结束。对于代码大于 127 的字符，它们的前面将会被加上 `M-` （表示「meta」），这与其它系统中在字符前面加上 `Alt-` 相当。
- `-E` 在每一行的结束处添加*美元符*（ `$` ）。

## 使用 cat 显示非打印字符

```bash
$ cat -t /etc/X11/XF86Config
...
# Multiple FontPath entries are allowed (they are concatenated together)
# By default, Red Hat 6.0 and later now use a font server independent of
# the X server to render fonts.
^IFontPath^I"/usr/X11R6/lib/X11/fonts/TrueType"
^IFontPath^I"unix/:7100"
EndSection
...
$ cat -E /etc/X11/XF86Config
...
# Multiple FontPath entries are allowed (they are concatenated together)$
# By default, Red Hat 6.0 and later now use a font server independent of$
# the X server to render fonts.$
$
    FontPath   "/usr/X11R6/lib/X11/fonts/TrueType"$
    FontPath   "unix/:7100"$
$
EndSection$
...
$ cat -v /etc/X11/XF86Config
...
^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@M-|M-8^X^@^@^@
P^@^O"M-X^O M-@^M^@^@^@M-^@^O"M-@M-k^@M-8*^@
@M-^H$M-@M-9|A(M-@)M-yM-|M-sM-*M-hW^A^@^@j^@
M-|M-sM-%1M-@M-9^@^B^@^@M-sM-+fM-^A= ^@ ^@
F^@^@   ^@M-9^@^H^@^@M-sM-$M-G^E(l!M-@M-^?
^IM-A5^@^@^D^@PM-^]M-^\X1M-H%^@^@^D^@tyM-G
```