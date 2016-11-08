title: Sublime Text 3 配置同步
date: 2016/4/9 00:48:00
tag:
	- Sublime Text

---

你是否有过Sublime Text 3 在各台机器下的配置不能同步而带来的种种困扰？ 你是否想过能够随便打开一台电脑都有自己熟悉的Editor环境？下面我利用官方推荐的方法，教大家同步自己的配置。

> Sublime新人们，我已经为你们打造了一个包含一些基本（主要）插件的配置环境，可以方便`Java`、`Python`以及`前端`开发人员使用，具体使用的插件请详见后面。

<!--more-->

## 部署教程

### Fork我的仓库

[点击进入仓库地址](https://coding.net/u/quentinchen/p/User/git)

把我的仓库fork到你自己的仓库（coding.net或者github都是可以的）里面去，以后就在自己的仓库下进行`push`和`pull`操作保持配置的同步了。

> 不太熟悉Git的小伙伴，如果想偷个懒直接使用我的配置，可以直接点击仓库里的[下载](https://coding.net/u/quentinchen/p/User/git/archive/master)来代替后面的`git clone`部分。

### 安装 ST3

首先到[官网](https://www.sublimetext.com/3)下载最新版的Sublime Text 3。

> 注意：Sublime Text 2 的操作会有不同，因为ST3各方面都超过了ST2，如果不是因为某些特殊的插件必须使用在ST2下的话，建议大家都直接使用ST3。目前为止ST3依旧采用无限免费试用的策略，而购买一个license的价格是70美元。

### 安装Package Control

打开Sublime Text 3，在主界面下按`ctrl+~`（反撇）（Windows）或者`command+~`（Mac OS X），在底部出现的command panel中键入：

```
import urllib.request,os,hashlib; h = '2915d1851351e5ee549c20394736b442' + '8bc59f460fa1548d1514676163dafc88'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

等待Package Control安装完成。

在主界面下按`ctrl+shift+p`（Windows）或者`command+shift+p`（Mac OS X），键入`pci`，如果出现package control: install，说明已经安装完成。

### 同步设置

在主界面菜单栏单击`Preference->Browse Packages`打开配置文件夹，删除目录下的`User`文件夹，在目录下执行Git命令（Mac或者Linux用户使用terminal，Windows用户使用Git Bash）

```git
git clone https://your_repo/User.git
//此处的https://your_repo是你fork我的仓库过去之后的仓库地址
//可以是https协议的，也可以是ssh协议的
```

重启ST3(重启之后由于各个插件下载的顺序不一致，可能会出现一些奇怪的错误，不用担心，等插件下载完毕之后再次重启ST3即可)即可完成和我的仓库配置同步。

### 关于配色

配色我使用了当先十分流行的SpaceGray的dark模式，由于我编辑了BracketHighlighter插件的配色，所以需要将`User`目录下的`Theme - Spacegray.sublime-package`文件复制到与`Packages`目录同级的`Installed Packages`目录下，这样，括号的配色也就实现了。

为了使用我的主题配色和括号匹配配色，请在菜单栏单击`preference->Color Scheme->Theme - Spacegray->base16-ocean.dark`以选择配色。

> Windows用户最好将字体换成`consolas`，我在Mac下使用的字体是`Monaco`。

SpaceGray配色如下：

![](https://packagecontrol.io/readmes/img/d0545a784bcadd0df626c55ef74954eeffc44b63.png)

BracketHighlighter 括号高亮的配色如下：

![](https://packagecontrol.io/readmes/img/2c23129492d6d74b8f9139711578e9ad0d1115a0.png)

## 插件说明

目前为止，我在ST3上安装的插件有：

- Anaconda：Python IDE
- BracketHighlighter：括号匹配
- Codecs33
- Color Highlighter：css颜色高亮
- ColorPicker：取色器
- ConvertToUTF8：将中文韩文等转换为UTF-8编码
- DocBlockr：块注释
- Emmet：前段代码展开工具
- GitGutter：支持代码改动标识
- MarkdownEditing：Markdown语法高亮
- Package Control
- SideBarEnhancements：文件夹侧边增强
- Theme - Spacegray