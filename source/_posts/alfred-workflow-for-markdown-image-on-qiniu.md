title: 利用Alfred Workflow快速上传粘贴板图片至七牛图床并在Markdown中引用 
date: 2016/3/19 22:51:00
tag:
	- Alfred 
	- 七牛 
	- 图床 
	- Markdown

---

## 应用场景

Markdown最大的缺陷就是不能方便的在文章中插入本地图片，所以通常情况下我们需要一个好的图床来帮助我们，国内现在使用体验最佳的图床就是七牛云存储，但是为了插入一张图片我们通常需要做的事情是：

- 截个图存在桌面
- 打开七牛云存储的网站，上传图片
- 复制图片地址
- 在Markdown中使用`![]()`语法调用图片插入
- 填写图片地址
- 如果你使用原图保护功能，还需要自己添加样式符

为了一张插入图片，真是心力交瘁。如果插入量巨大，真是不堪重负。所以我们需要使用一个工作流来一键帮我们完成这些复杂的机械化的工作。本教程**实现的目标**是：

- 截图到粘贴板
- 快捷键插入到Markdown文本

中途甚至不需要任何与编辑文本无关的工作，让你专心写作！

<!-- more -->

## 教程

### 准备工作

因为本方法用到了[Alfred](https://www.alfredapp.com)的PowerPackage扩展功能，所以你需要首先内（po）购（jie）其高级功能。

然后另一个准备工作就是七牛云存储了，到七牛云存储的[官网](https://portal.qiniu.com/signup?code=3lj4rde88jpzm)注册一个账号，开始使用。关于七牛云存储的一些设置，在这里我想说一下：

### 七牛云存储的设置

在你的空间的**数据处理**中应该配置好「图片样式」：新建图片样式。

![style](http://data.kchen.cc/4be39d7cf43e26849216aa2a61709d8c.png-960.jpg "图片样式")

使用图片样式的好处是你可以根据需求插入不同大小的图片，毕竟**Markdown是没有图片编辑和调整功能**的。我设置了如下的样式：

![style](http://data.kchen.cc/dbf2f0ee3a25351f3b2d8490a155eff8.png-960.jpg "图片样式")

这样我可以通过样式分割符合样式`-480.jpg`、`-960.jpg`、`-1920.jpg`调用不同大小的图片插入到文章中了。

### 配置七牛云存储文件同步

> **QRSync**是七牛云存储提供的同步上传客户端工具，可以用于Linux、Mac OS X、Windows等操作系统。使用QRSync，可将用户本地某个目录的所有文件同步上传到七牛云存储中，同时支持增量上传，可以只将目录中新增的文件上传至七牛云存储。 

首先，下载QRSBox

- Mac OS X: [qrsynccli darwin_amd64](http://devtools.qiniu.com/qiniu-devtools-darwin_amd64-current.tar.gz)
- Linux 64bits: [qrsyncli linux_amd64](http://devtools.qiniu.com/qiniu-devtools-linux_amd64-current.tar.gz)
- Linux 32bits: [qrsyncli linux_386](http://devtools.qiniu.com/qiniu-devtools-linux_386-current.tar.gz)

然后在你希望同步的文件夹下创建以下两个目录

- `CLI` ： 用于存放QRSync命令脚本
- `Data` ： 用于存放需要同步的图片、文件等

把下载好的QRSync解压后所有文件放到`CLI`目录下，在`CLI`目录下新建`conf.json`文件包含以下内容：

```json
{
    "src": "/home/your/sync_dir/Data",
    "dest": "qiniu:access_key=<AccessKey>&secret_key=<SecretKey>&bucket=<Bucket>&key_prefix=<KeyPrefix>",
    "debug_level": 1
}
```

其中：

- `src`是本地的同步目录`Data`，该目录下的文件会随时同步上传至七牛云存储。
- `AccessKey` 和 `SecretKey` 需要在七牛云存储平台上申请
    1. 开通[七牛开发者帐号](https://portal.qiniu.com/signup)
    2. 登录七牛开发者平台，[查看 Access Key 和 Secret Key](https://portal.qiniu.com/user/key)
- `<Bucket>` 是保存同步文件的资源空间名。

- `<KeyPrefix>` 是文件前缀，可选。如果设置了该参数，那么上传的文件名前都会加上前缀。这个前缀主要用于在空间中区分不同上传来源的文件。

配置完成后，在`CLI`目录下就可以随时使用如下命令来同步文件夹了：

```bash
./qrsync conf.json
```

> 另一个可以使用的工具是**QRSBox**，其支持后台实时监控目录，当有新文件加入到同步目录的时候，就会自动上传到相应的空间。但截止到我测试时（2016年3月19日）QRSBox仍有一些Bug导致工作流无法工作，所以我选择了更加可靠的QRSync。

### 配置WorkFlow

下面我们来配置WorkFlow工作流来让一切变得自动化起来，首先打开Alfred，进入Workflow，并且创建一个空白的工作流：

![](http://data.kchen.cc/b7ca368796817f6641eb530779170cb9.png-960.jpg)

然后添加一个热键，我选择的是`⌘`+`⇧`+`V`，这样和我的Annotate截屏快捷键`⌘`+`⇧`+`A`正好形成一对：

![hotkey](http://data.kchen.cc/78fcbef093b58a4b5d7d559414087555.png-960.jpg "HotKey")

然后添加一个action，选择`osascript`作为脚本语言

![](http://data.kchen.cc/ce75233c28ac53e40fa1905f6834918d.png-960.jpg)

我们添加如下脚本：

```applescript
property fileTypes : {¬
	{«class PNGf», ".png"}, ¬
	{JPEG picture, ".jpg"}}

on getType()
	repeat with aType in fileTypes
		repeat with theInfo in (clipboard info)
			if (first item of theInfo) is equal to (first item of aType) then return aType
		end repeat
	end repeat
	return missing value
end getType

set theType to getType()

if theType is not missing value then
	set filePath to "/Users/quentin/Documents/Qiniu/Data/" --这里换成你自己放置图片的路径
	set fileName to do shell script "date \"+%Y%m%d%H%M%S\" | md5" --用当前时间的md5值做文件名
	if fileName does not end with (second item of theType) then set fileName to (fileName & second item of theType as text)
	set markdownUrl to "![](http://data.kchen.cc/" & fileName & "-960.jpg)" --这里是你的七牛域名和设置的图片样式
	set filePath to filePath & fileName
	
	try
		set imageFile to (open for access filePath with write permission)
		set eof imageFile to 0
		write (the clipboard as (first item of theType)) to imageFile
		close access imageFile
		set the clipboard to markdownUrl
		try
			tell application "System Events"
				keystroke "v" using command down
			end tell
		end try
		do shell script "/Users/quentin/Documents/Qiniu/CLI/qrsync /Users/quentin/Documents/Qiniu/CLI/conf.json" --此处是你的QRSync脚本目录和命令
	on error
		try
			close access imageFile
		end try
		return ""
	end try
else
	return ""
end if
```

代码中有**4处需要替换修改地方**：

```applescript
set filePath to "/Users/quentin/Documents/Qiniu/Data/"
```

换成你自己设定的同步目录。

```applescript
set markdownUrl to "![](http://data.kchen.cc/" & fileName & "-960.jpg)"
```

设定成你的七牛空间域名：

![](http://data.kchen.cc/fbb46f61dcd1b87b7328b939cf79c6ee.png-960.jpg)

注意如果你在设置QRSync时预设了`<KeyPrefix>`前缀，记得在域名后面补上。另外再加上自己设定的图片样式`-960.jpg`，我在样式中自带了一个统一的文件后缀`.jpg`是为了让Markdown编辑器知道这是一个图片链接。

```applescript
do shell script "/Users/quentin/Documents/Qiniu/CLI/qrsync /Users/quentin/Documents/Qiniu/CLI/conf.json"
```

换成你的QRSync的命令脚本目录即可。

然后，添加一个通知，让我们得到上传成功的反馈：

![](http://data.kchen.cc/1fd5378d66501b4039dfb49e8027a94b.png-960.jpg)

最后，把Trigger、Action和Notification用线连起来，就大功告成了：

![](http://data.kchen.cc/8134b421b50a8cc1ef3901bc7d560a52.png-960.jpg)

现在，你只需要

### 额外的同步工作流

此外，我再提供一个额外的工作流来负责同步目录到七牛云存储，针对那些直接复制到同步目录下的文件。另建一个Workflow，如图所示：

![](http://data.kchen.cc/53b405507e35f3ad39a96d62c20370d2.png-960.jpg)

使用你喜欢的HotKey来启动上传，然后使用bash语言键入以下script：

```bash
/Users/quentin/Documents/Qiniu/CLI/qrsync /Users/quentin/Documents/Qiniu/CLI/conf.json
```

同样，把相应目录改成你自己的QRSync命令脚本目录就可以了。

**注意**：当上一个工作流没有成功上传时，可以使用这个工作流再次上传。

### 直接下载

我把自己写好的WorkFlow上传到百度云了，你也可以自己下载，然后修改参数直接使用，传送门：

[百度网盘下载](http://pan.baidu.com/s/1mhv4qTE)  密码：q3cd