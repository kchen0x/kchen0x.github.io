title: 八卦阴阳鱼，谈谈 Solarized 配色
date: 2016/11/16 21:02:07
updated: 
tags:
    - Editor
    - Solarized
---

![fish](http://ethanschoonover.com/solarized/img/solarized-yinyang.png)

相信长期使用 Terminal 的人对 Solarized 这个配色都不会陌生，许多的编辑器和终端都自带这个配色。今天想谈谈这个配色是因为，我觉得 Solarized 配色的作者的理念和这套配色的独特之处很让人觉得惊奇。

<!--more-->

## 八卦阴阳鱼

为什么我把 Solarized 称为「八卦阴阳鱼」呢？是因为这套配色设计之初就使用同一套配色适配了明暗两种主题，两种主题下，相同配色的文字都力求最大程度的和谐，这从上面这张官网示例图就可以看得出来，作者以八卦图展示，我觉得无论从效果上，还是从精神理念上都十分贴切。

Solarized 作为 16 色 ASNI 配色，还同时向上兼容 256 配色和向下兼容 5 配色。之所以说这套配色很独特，其实体现在几个方面。

第一，他是非传统的 16 色配色方案。

传统的 ASNI 16 色配色从 1~16 分别是黑、红、绿、黄、蓝、紫、青、白 8 种颜色以及对应的亮黑、亮红、亮绿、亮黄、亮蓝、亮紫、亮青、亮白 8 种亮色组成。可 Solarized 另辟蹊径，它增加了橙和靛两色，形成了 8 种基色着重色，而把 8 种亮色改为了明暗两种背景色调和 4 种内容色调，共 8 个阶梯色：

![color-tones](http://ethanschoonover.com/solarized/img/solarized-palette.png)

作者强调说自己的颜色选择严格按照 CIELAB 的色彩明度关系，同时选择根据色轮关系精调的色度组合，已经在实际生活中经过了多个标准测试。什么是标准测试且不管，在 Solarized 配色下的文字是目前为止我看起来眼睛最舒服的配色之一，这是不争的事实。

![contrast](http://ethanschoonover.com/solarized/img/solarized-vim.png)

## 配色特性

### 精心选择的对比度

想必大家都有过这样的经验，在一个晴朗的午后去户外、或者落地窗旁捧着一本书细细品味是一件非常舒服惬意的事情。当然不能直接坐在阳光下，那样的话光线太强烈，我们坐在一片树荫下，这时书本和文字的对比度很养眼，Solarized 正是力求模拟这样养眼的对比度。

![contrast](http://ethanschoonover.com/solarized/img/solarized-selcon.png)

最重要的是，Solarized 在降低亮度对比度的同时，尽量的保留了色调的对比度，大大提高了语法高亮的可阅读性。

### 明暗随心调

正如前面所言，Solarized 使用同一套配色来同时适配明暗两个主题，根据心情和场合的切换简直不能更方便。

![both-sides](http://ethanschoonover.com/solarized/img/solarized-dualmode.png)

### 向下兼容

前面还提到了，Solarized 提供 5 色的向下兼容，可以选择 4 个阶梯色加上一个着重色就可以用在很多的 Web 设计和测试上。

![5-palette](http://ethanschoonover.com/solarized/img/solarized-165.png)

### 轴对称特性

![symmetry](http://ethanschoonover.com/solarized/img/solarized-sym.png)

从上图可以看出，在同一套阶梯色上，不论是 dark 模式还是 light 模式，二者向对方过渡，都具有轴对称的明度对比度变化。

## 使用

Solarized 配色的使用范围很广：

- IDE： IntelliJ Idea、 Eclipse
- Editor： Sublime Text、 VIM、 Emacs
- Terminal Emulator： Terminal、 iTerm

还有网站和各种出版物等都可以使用。

![screenshot](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/5fbcd923003434962c387ad1b43a6939.png-960.jpg)

这是我在我在 iTerm2、Tmux 和 Vim 中搭配使用的 Solarized dark 配色。

以后有时间介绍一下这些软件的配置设置。