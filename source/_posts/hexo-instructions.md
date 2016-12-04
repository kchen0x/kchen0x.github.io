title: 基于 Hexo 的全自动博客构建部署系统
date: 2016/11/12 11:21:03
updated: 
tags:
    - Hexo
    - Travis
---

> 在互联网渗透到个人，自媒体信息爆炸的今天，大家都希望有一个归自己所有、掌控的**个人博客**。因为无论是记录生活，还是分享技术，自定义博客都可以按照自己的要求，为自己提供熨帖的服务，而不必受到条条框框的限制。但是普通用户又囿于没有高超的编程技术和前端本领，而在自定义博客面前望而却步了。本文的目标就是通过介绍 Hexo+Github/Coding.net+Travis+CloudXNS 形成一套完整的个人博客自动构建、部署、国内国外双线路负载均衡的系统。真正做到**随时随地写文章，一键部署到网站**的全自动化流水线。

## 什么是 Hexo

> Hexo is a fast, simple & powerful blog framework.

[Hexo] 是一个极速、简单且强大的静态博客架构。它使用 Node.js 作为构建引擎，上百个文件在几秒钟内便可构建完成；而且拥有着丰富的插件库，因开源而显得生机勃勃，可扩展性很好；最重要的，它支持 [Markdown] 作为书写语言，极大地方便了博客的撰写。

<!--more-->

### 安装 Hexo 

Hexo 的安装和使用都非常便捷。但是，你的计算机环境中必须有这两个东西：

- [Node.js]
- [Git]

其中 Node.js 是 Hexo 的构建引擎，Git 是版本控制工具，关于二者的介绍和安装我就不在这里赘述了。Mac 用户推荐使用 [Homebrew] 进行安装。安装完成后，我们在终端中键入以下命令确认安装结果：

```bash
node -v
git --version
```

如果你的环境没有问题的话，那么我们就可以安装 Hexo 了：

```bash
npm install hexo-cli -g
```

安装完成后，进入一个目录作为存放博客资源的目录，然后：

```bash
hexo init blog
cd blog
npm install
hexo server
```

不出意外的话，终端中输出这样的结果

```bash
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

那么恭喜，你的博客已经构建成功了！你可以到输出结果指定的地址去访问你的博客了。

![hexo-init](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/d3fcdb56c385052471d97e8a7b7d3a02.png-960.jpg)

### 撰写博文

撰写博文也非常简单，在你的博客目录 `blog` 下键入命令：

```bash
hexo new post <title>
```

就可以新建一篇博文了，如果你想写一篇草稿，而不发布的话，那么你可以键入：

```bash
hexo new draft <title>
```

首先我们来创建一篇名为 `hello` 的博文吧。

```bash
hexo new post hello
```

生成的源文件路径是：

```bash
INFO  Created: <blog-dir>/source/_posts/hello.md
```

我们可以用任何的编辑器打开这个文件。

```yaml
---
title: hello
date: 2016-11-12 12:39:20
tags:
---

```

这是 `post` 模板自动帮我生成的 `yaml` 文件头。其中 `title` 是博文的标题，我们可以改成「你好！」；`tags` 是博文的标签，我们可以加上「demo」、「hello」两个标签，从第 7 行开始我们就可以撰写博客的正文了。假设我写好的博客如下：

```markdown
---
title: 你好
date: 2016-11-12 12:39:20
tags:
    - demo
    - hello
---

欢迎来到 [Hexo](https://hexo.io/)！这是你的第一篇博文。

## 快速开始

### 创建一篇新博文

$ hexo new post "My New Post"

更多信息……

## 结语

谢谢你使用 Hexo。
```

然后保存这篇文章，之后访问 http://localhost:4000 你便可以看到这篇文章已经发布到你的博客中了（注意：`hexo server` 需要保持运行才能访问）。是不是非常的简单？

![first-post](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/96632da40c3f060c59e09c37b842bc3f.png-960.jpg)

更多信息，请查阅 Hexo 的[官方文档]。

## 博客部署

上文提到的博客，其实是通过 `hexo server` 搭建在本地的一个临时服务器，只有自己通过 http://localhost:4000 才能访问到，如果你想要把自己的博客发布的互联网上，你需要利用一个服务器来部署你的博客。

Hexo 是基于静态页面的博客系统，意味着对服务器的要求可以非常的低。低到什么程度呢？[Github] 和 [Coding.net] 就提供了免费的静态页面的托管服务，我们也就正好可以利用这个特性把我们的博客部署到这两个网站上面（为互联网的免费精神干杯🍻）。

> 我比较推荐把博客部署到 Github 上面，但是由于我国高墙的缘故，国内访问 Github 的速度有限，好在国内也有非常优秀的 Git 托管网站 Coding.net。所以国内用户可以非常方便的使用他家的免费服务来托管自己的博客，另外，我在后文还会介绍同时部署自己的博客到两个网站，然后利用 DNS 负载均衡技术来让国内国外访问分走不同线路实现全网加速，这是进阶教程了，我们先按下不表。

### 创建托管仓库

为了普适性和走国际化线路，本文以 Github 作为例子来介绍部署托管的流程。

我们用来托管博客的服务叫做 [Github Pages]，它是 Github 用来提供给个人/组织或者项目的网页服务，只需要部署到你的 Github Repository，推送代码，便可以实时呈现。

首先，你需要有一个 Github 的账号。然后创建一个名称为 `<username>.github.io` 的仓库来托管网页即可。

以我的 Github 为例，我的用户名是 `quentin-chen`，所以创建一个名为 `quentin-chen.github.io` 的仓库，创建的仓库地址便是：https://github.com/quentin-chen/quentin-chen.github.io.git 创建完后，我们可以暂时不用管它，不需要往仓库里面 `push` 任何的东西。

### Hexo 部署配置

接着，我们来配置一下本地的 Hexo。

在博客的根目录下有一个名为 `_config.yml` 的文件，这是博客的主配置文件。前面的其他部分我们先不理会，后文再谈，我们先看最后的 `Deployment` 配置项：

```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type:
```

根据官方的文档显示，现在 Hexo 支持 Git、Heroku、Rsync、OpenShift、FTPSync 等部署方式，我们选择 Git 来部署的话，需要首先安装 hexo-deployer-git 插件：

```bash
npm install hexo-deployer-git --save
```

然后编辑上面的配置文件

```yaml
deploy:
  type: git
  repo: <repository url>
  branch: [branch]
  message: [message]
```

我们需要把刚才创建的仓库地址添加进来，`branch` 和 `message` 项可以不填，默认情况下推送到 `master` 分支，这里我建议使用 SSH 加密的仓库地址（参看 [Github 官方文档]配置 SSH 免密操作）。

保存配置文件之后，我们在博客的跟目录键入：

```bash
hexo generate
hexo deploy
```

上述命令可以简化成

```bash
hexo g -d
```

便可以把博客部署到 Github 了。现在，所有人都可以通过 `http://<username>.github.io` 来访问自己的博客。

## 博客配置

前两章我们已经了解了 Hexo 的安装使用和如何部署自己的博客到 Github，现在我们来自定义一下我们博客。首先，主要的配置都在博客根目录的 `_config.yml` 文件里：

```yaml
# Site
title: Hexo
subtitle:
description:
author: John Doe
language:
timezone:
```

这一组配置是博客的描述，`title` 和 `subtitle` 分别是博客的站名，和副标题，`description` 对搜索引擎收录博客会有帮助。在多数主题中，一般只有标题和副标题会显示（我的主站只显示标题）：

![title](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/cbb658e56aaa0004500447f9fafc3309.png-960.jpg)

`language` 一般可以选配 `en` 或者 `zh-cn`，分别是英文和简体中文。

```yaml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```

这个部分需要修改的只有 `url` 选项，改成博客地址行了。后面的链接生成规则，保持不变即可。

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape
```

这一组配置是用来定义主题和配置插件的，可以看到，官方默认的主题叫做 `landscape`，就是上面的那个那个天际地平线的主题了。Hexo 的社区非常非常的活跃，[官方列表]已经收录了 84 个制作精美的主题（字母升序排列，热度不分先后），每个主题都有很大的差别和不一样的特性，大家可以自行去列表中浏览、预览、选择自己喜欢的主题，总有一款适合你的胃口（没有就自己改😄）。例如，我现在选择 [yelee] 主题：

```bash
git clone https://github.com/MOxFIVE/hexo-theme-yelee.git themes/yelee
```

把从官方列表中查到的主题仓库地址 `git clone` 下来，放到 `themes/` 下相应的目录中，然后我们修改博客根目录下的 `_config.yml` 文件

```yaml
theme: yelee
```

保存后，我们再次执行 `hexo server` 命令，预览一下网站的变化。

![yelee](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/469554ef7299916fe76ca677517130c1.png-960.jpg)

你看，主题是不是发生了巨大的改变，网站左侧现在显示的是 `author` 和 `sutitle` 了，右边的预览页面支持很多动态特效。每一个主题的详细配置参数都在 `/themes/<yourtheme>/_config.yml` 文件下，可以查阅该主题的官方页了解配置的解释。我在这里就不啰嗦了，当你选定了自己喜欢的主题，修改、配置好了之后。便可以使用、撰写、发布自己的博客了。

祝大家使用愉快，下面介绍 Hexo 的高阶教程。

---

## 域名和 DNS 负载均衡

### 绑定自定义域名

在前面的教程中，我们已经可以无碍的使用 Hexo 来写博客，可是有一个地方不太优雅，那就是域名。因为我们使用的是 Github 提供的二级域名 `github.io`，一来太长，二来也不太够自定义、个性化。关键是，你没有域名控制的自主权，DNS 和 CDN 都没法做，一切得看 Github 的网络环境。

所以，我们可以买一个域名，然后解析到 Github 去。关于选域名的问题，大家可以去看看[博客搬迁记 - K.Chen](http://kchen.cc/2016/11/04/new-beginning/)，像我现在选购的 `.cc` 的顶级域名，每年才 19 元。一口气买 5 年不过才一顿饭的花费。还有不不少顶级域名如 `.tech` 才每年 5 元，这价格跟白开水一样。实在不济，还有 `.tk`，`.ml`，`.cf`，`.ga` 四个**免费域名**提供开放注册（为互联网的免费精神干杯🍻）。域名在国内的万网、西部数码，国外的 Godady 等网站购买都是可以的。

购买域名之后，我们只需要把 DNS 解析创建两个 A 记录到：

```
192.30.252.153
192.30.252.154

如果使用 Coding.net，创建 CNAME 记录到：
pages.coding.me
```

之后在自己的博客根目录下创建一个 `CNAME` 文件，里面放你解析的域名。

之后修改根目录下的 `_config.yml` 文件，把站点地址更新成新的绑定的域名即可。

```yaml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com
```

现在，就可以使用自己的域名访问自己的博客了，例如： http://kchen.cc 

### DNS 负载均衡

为了让国内国外都顺畅地访问我们的博客，我们可以把自己的博客同时部署到 Github 和 Coding.net：

```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  - type: git
    repo: https://github.com/quentin-chen/quentin-chen.github.io.git,master
  - type: git
    repo: git@git.coding.net:quentinchen/quentinchen.git,master
```

可是问题来了：

> 我只有一个域名，怎么让不同地方的人访问的时候给他不同地方的资源呢？

其实，这是一个典型负载均衡问题，就是把访问流量分流提高访问流畅性，只不过我们在这里还有穿越国内高墙限制的用途在里面。常见的负载均衡技术有 CDN 和 DNS 均衡等。

CDN 的做法是把网站常访问数据缓存到 CDN 服务器，当用户访问网站的时候 DNS 会把请求发送到离用户最近的 CDN 服务器上去。听起来是不是很棒？但是 CDN 服务一来挺贵，当然也有免费的，可是很难做到全球网络范围内的 CDN，另一方面流量都是 CDN 去帮我们做了，如果想有一些客制化的要求不容易实现。何况，其实 Github 和 Coding.net 自己本身就做了挺好的 CDN 加速，我们既然把博客托管在上面，善加利用就是了。所以我们采用 DNS 负载均衡来实现国内外分流。

大部分人可能天天都用着 DNS 却不知道它的基本原理，你可能知道我们访问互联网需要查询 DNS 服务器：

![DNS](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/3bddb2d99dab6a63f0b3da0402af2bad.png-960.jpg)

可是你知道它具体是怎么工作的么？

其实 DNS 系统是一个典型的树状架构，上图所示的 DNS 服务器其实应该叫 DNS 缓存查询服务器，它是为了减轻互联网上 DNS 查询的负载所设计的。如果你的请求没有命中缓存，那么这个缓存服务器就会自己进行一次标准查询，然后再把结果缓存起来，简单来说就是从根服务器开始一级一级的问。我们以前经常谈到根服务器的重要性其实就体现在这里了，它保留了对所有域名的起始解释权。

> 我国高墙常使用的一个技术手段称为 DNS 投毒，就是在根服务器上投毒，把需要墙的网站域名解析到不存在的地址上去。再配合地方的 DNS 劫持，你就别想访问到这些被墙的网站了。

根据 DNS 的特点我们可以使用 [CloudXNS] 提供的免费智能 DNS 解析服务（再次为互联网的免费精神干杯🍻）来动态解析我们域名。

为什么使用 CloudXNS 呢，不仅因为它免费，而且：

- 不但拥有主流 DNS 产品的全部功能，而且扩展有自己独有的解析记录类型，在解析速度方面也明显优于国内其他同类产品，它的解析速度比传统 DNS 快两到三倍，DNS 配置生效更快，可做到实时生效。
- 有非常多的线路选择，不仅可以按照海内、海外分类，甚至海内可以按照移动、联通、电信、铁通等等再划分到省级线路，海外可以详细到不同的地区和国家等上百条不同的解析线路：

![routes](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/17b74c02dcce4490b23b55851181684e.png-960.jpg)

因为我们的博客始终是个小站，用不了那么多的线路区分，我们主要利用海内海外的线路来分流数据到 Github 和 Coding.net。

首先注册一个 CloudXNS 的账号，然后去原域名购买商处，将域名的 DNS 服务器指定为 CloudXNS 的服务器。

```
lv3ns1.ffdns.net
lv3ns2.ffdns.net
lv3ns3.ffdns.net
lv3ns4.ffdns.net
```

然后在 CloudXNS 设置你的域名，并添加如下域名解析：

![balance](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/0ac2ac1d019390aeb81baeb35196b482.png-960.jpg)

这样，默认访问将会被解析到 Coding.net，但是海外访问会被解析到 Github。非常方便。如果你的网站域名已经备案了的话，你还可以使用 CloudXNS 提供的免费 CDN 服务进一步为自己的网站提速。

---

## Travis CI 持续集成

到这一步，你其实已经可以很简便的使用 Hexo 来写博客了，可人是一种永远不满足的动物。使用 Hexo 来写博客有一个问题就是，我只能在安装了 Hexo 的计算机上写东西，然后 `hexo generate` 再 `hexo deploy` 到托管服务器。如果我想在没有安装 Hexo 的电脑上写博客呢？如果我想随时随地修改，或者写一下博客呢？

当然一个简单办法就是把自己博客的源文件也托管到 Github，每次都把源文件 `clone` 下来，然后安装 Hexo，再构建发布就好了。可是这样做的话，始终不太优雅。那可以搞一个远程的服务器，把构建博客的事情交给服务器，每次要写博客的时候连接到服务器上去就行了。可既然我们把博客都托管到了 Github，为甚么博客的构建还需要服务呢？不行，这样还是不够优雅。如果有什么服务能够代替上述的那个服务器就好了，答案是确实有的——**持续集成**。

### 什么是持续集成

> **持续集成**（Continuous Integration）是一种软件开发实践。 在持续集成中，团队成员频繁集成他们的工作成果，一般每人每天至少集成一次，也可以多次。 每次集成会经过自动构建（包括自动测试）的检验，以尽快发现集成错误。

### 什么是 Travis CI

> Travis CI 是目前新兴的开源持续集成构建项目，它与 jenkins，GO 的很明显的特别在于采用 yaml 格式，同时它是在线的服务，不像 jenkins 需要你本地搭建服务器，简洁清新独树一帜。目前大多数的 Github 项目都已经移入到 Travis CI 的构建队列中，据说Travis CI 每天运行超过 4000 次完整构建。对于做开源项目或者 Github 的使用者，如果你的项目还没有加入 Travis CI 构建队列，那么我真的想对你说 OUT 了。

### Travis 和 Hexo

为什么我们要选择 Travis 呢，因为它和 Github 集成的程度非常高，对于 Github 上的开源项目，可以免费在 Travis 上构建（我们是不是该为免费的互联网精神干杯🍻），而非开源的私有项目想在 Travis 上构建价格还是非常感人的。

我先把在 Travis 上进行自动构建的思路说一下：

![sequence](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/cbc37bcd05062212564fe2aaaee6631f.png-960.jpg)

- 我们在在 Github 的博客仓库里新建一个 `blog-source` 分支，然后把博客的源码托管到这个分支
- 每当我们在本地写好了博文之后，把修改 `push` 到该分支
- Travis 上可以对这个项目的 `blog-source` 分支设置钩子，每当检测到 `push` 的时候就去仓库 `clone` 源代码
- Travis 执行构建脚本
- Travis 把构建结果通过 `push` 部署到 `master` 分支或者 Coding.net 的仓库里 

在这样的自动化流程下，我们唯一需要做的事情就是 `push` 我们的博文到源代码分支，其他的事情交给 Travis，当然，这一流程是需要我们配置的。

### 配置 Travis

#### 注册 Travis

Travis CI 不需要单独注册，直接使用 GitHub 账号登录就可以了。

上[官网]会发现有 Sign in with GitHub（使用GitHUb登录）和 Sign Up（注册），其实这俩做的事情都一样，就是用 GitHub 账号登录。登录后界面会显示你的 GitHub Repository，默认全部全部没有勾选，选择你的博客的 Repository 后完成第一步，如图：

![travis](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/c5c4ae39bc22681ccf481119befbe363.png-960.jpg)

如果你没有看到自己的项目，请点击右上角的 Sync with Github。

#### 安装 Travis CML

在进行下面的步骤之前，我们需要先安装 Travis 的 CML，因为后面的部署需要我们加密的自己的 SSH 私钥和 Github、Coding.net 通信。安装过程请看 [Travis CML Installation]：

首先必须有 [Ruby] 1.9.3 以上，检查了版本之后，安装 Travis，检查版本即可：

```bash
ruby -v
gem install travis -v 1.8.4 --no-rdoc --no-ri
travis version
```

如上，如果出现 1.8.2 这样的版本信息，则说明 Travis CI Command Line Client 安装成功。之后进行登录，执行：

```bash
travis login
```

按照提示登录就好了。

#### 配置 Travis

在博客根目录下添加 Travis CI 所需要的配置文件 `.travis.yml`，配置文件内容和一些说明如下：

```yaml
language: node_js
node_js: stable

# assign build branches
branches:
  only:
    - blog-source

# cache this directory
cache:
  directories:
    - node_modules

# S: Build Lifecycle
before_install:
  - openssl aes-256-cbc -K $encrypted_a0b7f0848317_key -iv $encrypted_a0b7f0848317_iv -in ./.travis/id_rsa.enc -out ~/.ssh/id_rsa -d
  - chmod 600 ~/.ssh/id_rsa
  - eval $(ssh-agent)
  - ssh-add ~/.ssh/id_rsa
  - cp .travis/ssh_config ~/.ssh/config
  - npm install -g hexo-cli # 安装 hexo
  - git clone -b mod https://github.com/quentin-chen/hexo-theme-even.git themes/even

install:
  - npm install # 安装 package.json 中的插件

script:
  - hexo generate

after_success:
  - git config --global user.name "Quentin_Chen"
  - git config --global user.email "quentin.chen@foxmail.com"
  - sed -i'' "/^ *repo/s~github\.com~${githubToken}@github.com~" _config.yml
  - hexo deploy
# E: Build LifeCycle
```

我逐步来讲解一下每一个配置项的意思。

```yaml
language: node_js
node_js: stable

# assign build branches
branches:
  only:
    - blog-source

# cache this directory
cache:
  directories:
    - node_modules
```

这里指定了构建的环境是 Node.js，版本是当前稳定版本。设置的 WebHook 钩子只检测 `blog-source` 分支的 `push` 变动。另外我们把 `node_modules` 文件夹放入缓存，这样可以大大节约每次构建的时间（2min -> 30s）。

```yaml
before_install:
  - openssl aes-256-cbc -K <you_key> -iv <your_iv> -in ./.travis/id_rsa.enc -out ~/.ssh/id_rsa -d
  - chmod 600 ~/.ssh/id_rsa
  - eval $(ssh-agent)
  - ssh-add ~/.ssh/id_rsa
  - cp .travis/ssh_config ~/.ssh/config
  - npm install -g hexo-cli # 安装 hexo
  - git clone <theme_repo> themes/<theme>
```

其实每次 Travis 构建的时候，相当于创建了一个干净的虚拟机，除了 Git 等必须的工具，甚至连 Node.js 的环境都是现安装的。所以我们在构建自己的博客之前，需要做一些环境的准备。

其中 2-6 行是用来配置 SSH 私钥的，这样才能让 Github 和 Coding.net 知道你的身份。这一部分我们后面再说。

第 7 行是在 Travis 中安装 Hexo 环境，第 8 行是安装主题。

> 关于主题这里，如果你对自己的主题做了修改（包括配置文件），那么应该把自己修改过的主题托管到 Github，这里填的 `<theme_repo>` 应是你自己地址。

```yaml
install:
  - npm install # 安装 package.json 中的插件

script:
  - hexo generate

after_success:
  - git config --global user.name "<You Name>"
  - git config --global user.email "<email>"
  - hexo deploy
```

这一部分，就是在 Travis 上模拟部署过程。因为要使用 Git，所以要提前配置好 Committer 的信息。

#### 生成私钥加密文件

> 什么是私钥？

私钥就是密钥对（密钥对指一对公钥和私钥），我们在使用 Github 的时候，首先需要在 Github 上配置公钥，这是最基础的。那么，存在我们本地的私钥就是你的个人身份标示，如果你的项目 git 地址配置的是 git@github.com:username/projectname.git（相对的还有 https://github.com/username/projectname.git）， 当你在对 Repository 在一些操作（比如 `push` 等），则需要私钥进行身份验证了（这是自动验证的，如果是使用 https 的配置，则需要提供用户名和密码）。

我们在 Travis CI 上自动部署代码，就牵扯到了 `push` 操作，那么就需要提供私钥了。

> 为什么生成私钥加密文件？

将私钥直接放在项目里，那么人人都能看到。私钥的泄露将会发生一系列的问题，比如有坏人拿你的私钥直接操作你的 git 项目，你能干啥他也能干啥（原理上面讲了），这咋整？我们需要对私钥进行加密。

Travis 提供了加密文件的支持，什么意思呢？我们可以对文件（这里指私钥）在本地进行加密，然后把加密过后的文件放在项目里，那么别人就无法获取里面的真实内容。然后我们在让 Travis 执行脚本的时候，在读取加密文件之前对文件进行解密（使用的解密密码提前在 Travis 上配置好了），这样就可以达到不将文件内容暴露，并且让 Travis 获取到真实内容的目的了，大概的时序图如下：

![enc-key](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/8801dc034d298600bfc9b7124a0f42cd.png-960.jpg)

开始吧，我们首先把自己的在博客的根目录下建立 `.travis` 文件夹来存放相关的资料：

```bash
mkdir .travis && cd .travis
```

把本地的私钥复制一份过来，用 Travis 加密，然后删除（**切记加密完了一定要删除原始密钥！！！**）：

```bash
cp ~/.ssh/id_rsa .
travis encrypt-file id_rsa
rm id_rsa
```

现在 `ls` 命令查看一下 `.travis` 目录应该只有 `id_rsa.enc` 这一个文件才对。然后我们再在当前目录下新建一个 `ssh_config` 用来配置 Travis 上的 SSH Client。

```
Host *
  User git
  StrictHostKeyChecking no
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes
```

现在，我们在 Travis 网站，博客项目的设置（项目右上角）里可以看到两个用于解密私钥的环境变量：

![travis-env](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/c180cfd5996f2b3c1d43f9017a11b7c9.png-960.jpg)

把这两个环境变量名复制到上面的 `.travis.yaml` 文件里替换相应部分：

```yaml
before_install:
  - openssl aes-256-cbc -K <you_key> -iv <your_iv> -in ./.travis/id_rsa.enc -out ~/.ssh/id_rsa -d
```

这样，全部的配置就完成了。

> Tips： Github 还支持 Application Token 的方式来认证身份，比使用 SSH 私钥要更简便，但考虑到 Coding.net 并不支持，我还是选择普适的方法。有兴趣的同学可以自己研究一下，就当做课后作业吧:D。

### 完成工作流

在进行工作流之前我们来检查一下我们现在工作目录和所有必须的东西：

```bash
.
├── .travis*
│   ├── id_rsa.enc
│   └── ssh_config
├── _config.yml*
├── db.json*
├── node_modules
├── package.json*
├── scaffolds*
├── source*
│   ├── CNAME*
│   ├── _posts
│   ├── about
│   ├── categories
│   ├── img
│   ├── media
│   └── tags
└── themes
```

我用星号标记的文件和文件夹都是十分重要的，确保 Git 覆盖了这些文件和目录，然后我们把目录 `push` 到 `github/blog-source` 仓库分支。Travis WebHook 立马就会检测到 `push`，然后开始构建了：

![build-success](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/e0f11bcfb411c5b73bdcac015ee87b3d.png-960.jpg)

上图是一次成功的构建，我们可以点开 `Job log` 看一下构建过程的 Log 文件，特别是构建没有成功的话，我们更要仔细阅读，找准问题，对症下药。

构建成功以后再去刷新你的网页，是不是已经是最新的了呢？

## 总结

至此，这套教程就全部写完了。我们来回顾一下：

- Hexo 的安装和使用
- 配置博客和使用自定义主题
- 部署自己的博客到 Github 或者 Coding.net 等静态页面托管服务
- 利用 CloudXNS 对自己的域名进行动态解析，以加速国内国外的全网访问速度
- 利用 Travis 实现全自动部署

希望你能学会，并爱上用这样的方式来写作博客！一切只是刚刚开始，可以折腾的东西还有好多好多，祝你玩得愉快！

> 在操作过程中有什么疑问，欢迎在下方留言。大家一起交流讨论。

[Hexo]: https://hexo.io
[Markdown]: http://kchen.cc/2015/10/05/Markdown-Manual/#Markdown-简明语法
[Node.js]: https://nodejs.org/zh-cn/
[Git]: https://git-scm.com
[Homebrew]: http://brew.sh
[官方文档]: https://hexo.io/docs/
[Github]: https://github.com
[Coding.net]: https://coding.net
[Github Pages]: https://pages.github.com
[Github 官方文档]: https://help.github.com/articles/generating-an-ssh-key/
[CloudXNS]: https://www.cloudxns.net
[官网]: https://travis-ci.org/
[Travis CML Installation]: https://github.com/travis-ci/travis.rb#installation
[Ruby]: http://www.ruby-lang.org/en/downloads/
[官方列表]: https://hexo.io/themes/
[yelee]: https://github.com/MOxFIVE/hexo-theme-yelee