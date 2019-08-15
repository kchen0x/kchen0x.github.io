title: Hexo 里的脚注插件
date: 2016/11/10 22:02:19
updated: 
tags:
    -  Hexo
---

## Markdown 的脚注

Markdown 基本语法中并不包含脚注[^1](英文称为footnote，用于为正文补充注解（解释性加注）或标明被引用于正文或注解的数据源。一般，脚注会在文章内以符号或数字标示，然后在文章末端（也就是文章的「脚」），列出所有的补充、数据源的详情。脚注让编者补充细节之余，也不影响行文的聚焦，让版面显得更整齐。)语法，但是脚注作为一种常见的文本格式，对于文字编辑工作者，特别是像我这种喜欢插入引文的人而言，有着很大的使用需求。所以 [Multi-Markdown](https://github.com/fletcher/MultiMarkdown-5)[^2](MultiMarkdown, aka MMD, is a derivative of Markdown that adds new syntax features, such as footnotes, tables, and metadata.) 在其扩充语法集中增添了脚注的语法：

<script src="https://coding.net/u/kchen0x/p/resources/git/raw/master/footnotes.js"><!--这是一段 Gist 的代码片段（coding 托管）--></script>

<!--more-->

上面的语法经过语法渲染得到的结果就如下所示：

> this is a basic footnote[^3]
> here is an inline footnote[^4](inline footnote)
> and another one[^5]
> and another one[^6]
> this one with lot content[^7]

## 插件的安装和使用

MMD 的脚注语法得到广泛的传播和认可，大部分的 Markdown 编辑器现在都采用了该语法来渲染脚注。可是 Hexo 的默认渲染器是不支持脚注语法的，所以我写了这个简单的 [Reference 插件![GitHub stars](https://img.shields.io/github/stars/quentin-chen/hexo-reference.svg?style=social&label=Star)](https://github.com/quentin-chen/hexo-reference)来实现脚注的渲染。该插件已收录于 Hexo [官方插件页](https://hexo.io/plugins/)。插件的安装和使用非常的简单，只需要进入博客目录，然后安装：

```bash
npm install hexo-reference --save
```

便可在文章中使用相关语法插入脚注了。

## 扩展功能

因为我有时候并不喜欢通过点击引用编号来往跳转于正文和脚注之间，相较之下我对维基百科风格的悬浮提示非常喜欢，所以我就想利用 Tooltip 工具来实现正文内的脚注呈现。这里我选用了 [Hint.css🔗](https://kushagragour.in/lab/hint/) 提供的悬浮提示库，为所有的引用编号添加 css 属性，从而实现了一种更加优雅的脚注呈现方式。

你不妨把鼠标移动到编号上去看看效果😊。

[^3]: basic footnote content
[^5]: paragraph
footnote
content
[^6]: footnote content with some [markdown](https://en.wikipedia.org/wiki/Markdown)
[^7]: **Lorem Ipsum** is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book.
