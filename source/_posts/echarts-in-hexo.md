title: 在 Hexo 中引入 ECharts 图表
date: 2016/11/05 12:56:09
updated: 
tags:
    - Hexo
    - ECharts
---

> 图上的按钮可以点击交互🙂

<script type="text/javascript">
var xAxisData = [];
var data1 = [];
var data2 = [];
for (var i = 0; i < 100; i++) {
    xAxisData.push(i);
    data1.push((Math.sin(i / 5) * (i / 5 -10) + i / 6) * 5);
    data2.push((Math.cos(i / 5) * (i / 5 -10) + i / 6) * 5);
}
</script>

{% echarts 250 '100%' %}
option = {
    title: {
        text: ''
    },
    legend: {
        data: ['input', 'output'],
        align: 'left'
    },
    toolbox: {
        // y: 'bottom',
        feature: {
            magicType: {
                type: ['stack', 'tiled']
            }
        }
    },
    tooltip: {},
    xAxis: {
        data: xAxisData,
        silent: false,
        splitLine: {
            show: false
        }
    },
    yAxis: {
    },
    series: [{
        name: 'input',
        type: 'bar',
        data: data1,
        animationDelay: function (idx) {
            return idx * 10;
        }
    }, {
        name: 'output',
        type: 'bar',
        data: data2,
        animationDelay: function (idx) {
            return idx * 10 + 1000;
        }
    }],
    animationEasing: 'elasticOut',
    animationDelayUpdate: function (idx) {
        return idx * 5;
    }
};
{% endecharts %}

## 什么是 ECharts 图表

> [ECharts](http://echarts.baidu.com/index.html)，一个纯 Javascript 的图表库，可以流畅的运行在 PC 和移动设备上，兼容当前绝大部分浏览器（IE8/9/10/11，Chrome，Firefox，Safari等），底层依赖轻量级的 Canvas 类库 ZRender，提供直观，生动，可交互，可高度个性化定制的数据可视化图表。

<!--more-->

![ECharts](http://jquery-plugins.net/image/plugin/echarts-interactive-charting-library-by-baidu.png)

在 [12 Best Charting Libraries for Web Developers - Christopher Watkins](http://blog.udacity.com/2016/03/12-best-charting-libraries-for-web-developers.html)  一文中，博主 Christopher Watkins 向我们介绍了 12 款为网页开发者推荐的绘图库。其中不乏大名鼎鼎的 Google Charts，D3.js 和 HighCharts 这样的富图表库，还有不少其他的特型图表库，它们在展示某些特定的图表时，非常出色。百度开发维护的 Echarts 当然也在其中。

Echarts 作为国产工具，在语言上对中文开发者有着天然的优势，官方文档对每一个细节、参数、配置都有详尽的说明，对于新手非常的友好。另外一个重要的方面，就是 Echarts 的图表颜值很高，默认的主题和配色可以呈现出优雅漂亮的图表。所以，我也一直选择 Echarts 作为我的网页图表绘图工具。

## Hexo 中的 Echarts

Hexo 的 [Echarts 插件![GitHub stars](https://img.shields.io/github/stars/quentin-chen/hexo-tag-echarts3.svg?style=social&label=Star)](https://github.com/quentin-chen/hexo-tag-echarts3)是我根据周旅军的原型插件[^1]开发的，已收录于 Hexo [官方插件页](https://hexo.io/plugins/)。插件的安装和使用非常的简单，只需要进入博客目录，然后安装：

```bash
npm install hexo-tag-echarts3 --save
```

之后在文章内使用 ECharts 的 `tag` 就可以了：

```
{% echarts 400 '85%' %}
\\TODO option goes here
{% endecharts %}

```

其中 `echarts` 是标签名，不需要更改，`400` 是图表容器的高度，`85%` 是图表容器的相对宽度。而在 `tag` 之间的部分，则是需要自己填充的图表数据了。

我们来看一个使用样例：

```
{% echarts 400 '85%' %}
{
    tooltip : {
        trigger: 'axis',
        axisPointer : {            // 坐标轴指示器，坐标轴触发有效
            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
        }
    },
    legend: {
        data:['利润', '支出', '收入']
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    xAxis : [
        {
            type : 'value'
        }
    ],
    yAxis : [
        {
            type : 'category',
            axisTick : {show: false},
            data : ['周一','周二','周三','周四','周五','周六','周日']
        }
    ],
    series : [
        {
            name:'利润',
            type:'bar',
            itemStyle : {
                normal: {
                    label: {show: true, position: 'inside'}
                }
            },
            data:[200, 170, 240, 244, 200, 220, 210]
        },
        {
            name:'收入',
            type:'bar',
            stack: '总量',
            itemStyle: {
                normal: {
                    label : {show: true}
                }
            },
            data:[320, 302, 341, 374, 390, 450, 420]
        },
        {
            name:'支出',
            type:'bar',
            stack: '总量',
            itemStyle: {normal: {
                label : {show: true, position: 'left'}
            }},
            data:[-120, -132, -101, -134, -190, -230, -210]
        }
    ]
};
{% endecharts %}
```

上述代码渲染出来的 Echarts 图表如下：

{% echarts 400 '81%' %}
{
    tooltip : {
        trigger: 'axis',
        axisPointer : {            // 坐标轴指示器，坐标轴触发有效
            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
        }
    },
    legend: {
        data:['利润', '支出', '收入']
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    xAxis : [
        {
            type : 'value'
        }
    ],
    yAxis : [
        {
            type : 'category',
            axisTick : {show: false},
            data : ['周一','周二','周三','周四','周五','周六','周日']
        }
    ],
    series : [
        {
            name:'利润',
            type:'bar',
            itemStyle : {
                normal: {
                    label: {show: true, position: 'inside'}
                }
            },
            data:[200, 170, 240, 244, 200, 220, 210]
        },
        {
            name:'收入',
            type:'bar',
            stack: '总量',
            itemStyle: {
                normal: {
                    label : {show: true}
                }
            },
            data:[320, 302, 341, 374, 390, 450, 420]
        },
        {
            name:'支出',
            type:'bar',
            stack: '总量',
            itemStyle: {normal: {
                label : {show: true, position: 'left'}
            }},
            data:[-120, -132, -101, -134, -190, -230, -210]
        }
    ]
};
{% endecharts %}

可以看到，在插件的帮助下，在 Hexo 中使用 Echarts 图表是非常方便的，而且得到的图表质量也非常的好。

> 按照上例不能正确绘制图表的同学，请照下面的指导修改一下 ECharts 的模板文件。
> 用编辑器打开博客目录下 `node_modules/hexo-tag-echarts/echarts-template.html` 文件。
> 作如下修改：

```html
<div id="<%- id %>" style="width: <%- width %>;height: <%- height %>px;margin: 0 auto"></div>
<script type="text/javascript">
	...
</script>

将上面的部分 1、2 行之间添加一行改成下面的代码：

<div id="<%- id %>" style="width: <%- width %>;height: <%- height %>px;margin: 0 auto"></div>
<script src="http://echarts.baidu.com/dist/echarts.common.min.js"></script>
<script type="text/javascript">
	...
</script>
```

> 这样便可以在使用 Echarts `tag` 的时候载入 JavaScript 资源了。

## 如何用好 ECharts

ECharts 的官方文档详细的介绍了如何在开发中使用 ECharts，如果你是一名前端 JavaScript 开发者，当然可以仔细的阅读并用代码实现。但是，如果你只是希望在博客中插入一些简单的图表，那么从头到尾写一个完整的图表配置，甚至是 ECharts 生成代码是没有必要的，所以我们需要更简单的方法来生成相关的数据。[百度·图说]()是 ECharts 团队开发的另一款非常方便的工具，提供 UI 界面给你快速的绘制和定义图表，然后导出为代码、图片以及其他格式。

打开百度图说，点击**开始创建图表**，我们可以看到有非常多的选择：

![创建图表](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/ed9592d667cc0ad300cd455e87d5bafe.png-960.jpg)

我们随便创建某种类型的图表，然后便可以一边修改数据、调整参数、增加功能和工具，一边实时的看到图表效果，非常的直观：

![图表调整](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/278909667c73959629d785571fc0a96e.png-960.jpg)

然后我们可以通过点击**显示代码**得到 ECharts 的配置参数：

![配置参数](http://7xin49.com1.z0.glb.clouddn.com/mac_qrsync/ebe5dd074d1eaf2b305cf609ff43a988.png-960.jpg)

把这段配置参数拷贝复制到我们的博客里面，直接粘贴到 ECharts `tag` 里面，就可以了：

{% echarts 400 '81%' %}
{
    title: {
        text: "男性女性身高体重分布",
        subtext: "抽样调查来自: Heinz  2003"
    },
    tooltip: {
        trigger: "axis",
        showDelay: 0,
        axisPointer: {
            type: "cross",
            lineStyle: {
                type: "dashed",
                width: 1
            }
        }
    },
    legend: {
        bottom: 5,
        data: ["女性", "男性"]
    },
    toolbox: {
        show: true,
        feature: {
            mark: {
                show: true
            },
            dataZoom: {
                show: true
            }
        }
    },
    xAxis: [
        {
            type: "value",
            power: 1,
            precision: 2,
            scale: true
        }
    ],
    yAxis: [
        {
            type: "value",
            power: 1,
            precision: 2,
            scale: true
        }
    ],
    series: [
        {
            name: "女性",
            type: "scatter",
            data: [[161.2, 51.6], [172.9, 62.5], [153.4, 42], [160, 50], [147.2, 49.8], [168.2, 49.2], [175, 73.2], [157, 47.8], [167.6, 68.8], [159.5, 50.6], [175, 82.5], [166.8, 57.2], [176.5, 87.8], [170.2, 72.8], [174, 54.5], [173, 59.8], [179.9, 67.3], [170.5, 67.8], [162.6, 61.4]]
        },
        {
            name: "男性",
            type: "scatter",
            data: [[174, 65.6], [164.1, 55.2], [163, 57], [171.5, 61.4], [184.2, 76.8], [174, 86.8], [182, 72], [167, 64.6], [177.8, 74.8], [180.3, 93.2], [180.3, 82.7], [177.8, 58], [177.8, 79.5], [177.8, 78.6], [177.8, 71.8], [177.8, 72], [177.8, 81.8], [180.3, 83.2]]
        }
    ]
}
{% endecharts %}

> Tips: 右上角的**区域缩放**工具可以缩放图中的局部区域，试试看吧。

怎么样，是不是非常的方便。

[^1]: 周旅军的 Echarts 插件已不再维护，我的几个 pull request 都没有响应，所以才根据它的原型，重做了这个新的插件。