---
title: "用Angular复制了一个Vue3的起始模板项目"
subtitle: "对比一些数据"
date: 2023-02-25T21:13:18+08:00
draft: false
tags: ["angular", "vue"]
categories: ["frontend"]
toc: true
featuredImage: ""
featuredImagePreview: ""
summary: "Vue 3 + Vite 的初始化模板还挺好看的，所以我给弄了个Angular版本的。它们比较起来在各项数据上有啥区别？"
toc: true
autoCollapseToc: true
---

**太长，不看**： 代码仓库在[这里](https://github.com/HymanZHAN/ng-starter-demo).

**更新**: 线上项目运行在[这里](https://ng-starter-demo.netlify.app/).

以前博客也提到过，上一份工作中前端开发用的是 Vue 2。但是最近换工作了换到了 SAP，工作上只能用 SAP 自研的 SAPUI5（着实难用:upside_down_face:）。闲暇时光里总还是按捺不住想尝鲜，对 Vite 感兴趣也不是一天两天了，于是打算周末试试看 Vue3 的起始模板，体验体验。新建项目运行起来之后，发现页面还挺清爽，有点内容，排版也听好看，遂开始手痒想着造一个 Angular 同款，看看对比起来如何。于是便有了开头的项目和这篇没什么营养的文章。

{{< admonition warning "不要吵架" false >}}
这里没有踩一捧一的意图，仅仅是纯粹的数字对比。两个我都用两个我都爱，它们都是我的翅膀:hugs:（大雾）。
{{< /admonition >}}

## 打包大小

下面是一个打包后 JS 代码大小的对比图：

{{< echarts >}}
{
"tooltip": {
"trigger": "axis"
},

"legend": {
"data": ["Angular", "Vue"],
"top": "5%"
},

"xAxis": {
"type": "category",
"data": ["Uncompressed", "Compressed (brotli)"]
},

"yAxis": {
"type": "value",
"name":"Size (kB)"
},

"series": [

{
"name": "Angular",
"itemStyle": {"color": "#ee6666"},
"data": [268, 76.1],
"type": "bar"
},

{
"name": "Vue",
"itemStyle": {"color": "#91cc75"},
"data": [84.8, 30.4],
"type": "bar"
}

]
}
{{< / echarts >}}

这里要注意的是，两个应用都包括了路由，且初始加载大小并没有计算懒加载的“关于”页面。我的第一印象有两点：

- Vue 还是相当轻量级的。 这点在日常工作开发中其实已有体会，并且 Vue 3 相比 Vue 2 提供了更加 tree-shakable 的设计肯定是更为精简。不过像这样两个几乎一样的项目放在一起，两相比较，还是更为直观。
- 压缩之后，负载的绝对差值看起来没有压缩前那么突出。当然我还是希望有朝一日 Angular 在这种大小的应用上能够一步一步突破（压缩前） 200kB、100kB 的大关。

当然你可能会说：“但是下载下来的 JavaScript，浏览器不还是得解析执行吗？”唔…:thinking:真的是这样吗？

## Lighthouse

下载下来的 JavaScript 代码解析是肯定要解析的，但是可不一定都会执行，否则“懒加载”也无从谈起。关于这点，我们可以看看 Lighthouse 给出的信息。相较于性能评分，Lighthouse 有一个稍微没那么多人知道的小工具——JavaScript 树形图。你可以在 Lighthouse 的结果页面点击“查看树状图”进行查看。Angular 和 Vue 的树状图分别如下（该图没有涵盖 `styles.js`，因此和前面有点出入)：

{{< image src="treemap-angular-min.png" caption="Treemap - Angular" width="80%" >}}

{{< image src="treemap-vue3-min.png" caption="Treemap - Vue" width="80%" >}}

当我们把未执行的 JavaScript 纳入计算，就会得到下面的统计图：

{{< echarts >}}
{
"tooltip": {
"trigger": "axis"
},

"legend": {
"data": ["Angular", "Vue"],
"top": "5%"
},

"xAxis": {
"type": "category",
"data": ["Uncompressed", "Compressed (brotli)", "Unused"]
},

"yAxis": {
"type": "value",
"name":"Size (kB)"
},

"series": [

{
"name": "Angular",
"itemStyle": {"color": "#ee6666"},
"data": [268, 76.1, 90.94],
"type": "bar"
},

{
"name": "Vue",
"itemStyle": {"color": "#91cc75"},
"data": [84.8, 30.4, 23.67],
"type": "bar"
}

]
}
{{< / echarts >}}

由此，个人的第三点印象:

- Angular 有大概 35% 的 JavaScript 未经执行，而 Vue 有大概 29%。 我没有仔细研究过，但我盲猜应该有相当一部分是在各自的路由模块中。而且没记错的话 Angular 的路由模块还是蛮大的…对比之下还是 Angular 的进步空间更大一些哈哈:smile:

那么大家最关心的 Lighthouse 分数本身怎么样呢？废话不多说，上图:

{{< image  src="lighthouse-side-by-side.png" caption="Lighthouse Results" width="100%" >}}

两个框架都达到了 100 分！🥳 不过这么小的一个应用没有 100 分反而不太说得过去哈哈。总体来说，Vue 在每个项目上都快上了 0.3 到 0.4 秒。自己尝试的时候记得要用隐私模式并把插件都关干净，否则会影响测试结果。

## 总结

很直白的对比，也没啥可总结的:joy: 在写代码的时候倒确实被勾起了关于宿主元素（host element）的回忆。简单来说，一个宿主元素就是一个 Angular 组件实例对应生成的 DOM 元素，并且这个 DOM 元素与其组件选择器保持一致。也就是说，同样的页面和组建布局，Vue 生成 HTML 会是这样，比较“纯净”：

{{< highlight html>}}

<main>
    <div class="item">
        <i>
            <svg></svg>
        </i>
        <div class="details">
            <h3>Documentation</h3>
        </div>
    </div>
</main>

{{< / highlight >}}

而 Angular 生成的 HTML 则会是这样（留意高亮的部分）:

{{< highlight html "linenos=table,hl_lines=2 3 6 8 14 15">}}

<main>
    <app-welcome>
        <app-welcome-item>
            <div class="item">
                <i>
                    <app-icon-documentation>
                        <svg></svg>
                    </app-icon-documentation>
                </i>
                <div class="details">
                    <h3>Documentation</h3>
                </div>
            </div>
        </app-welcome-item>
    </app-welcome>
</main>

{{< / highlight >}}

所以在迁移样式的过程中，有的 CSS 属性就不能原样保留在 CSS 的类选择器上，得放到[:host 选择器](https://angular.cn/guide/component-styles#host)里。

扯开点说，对于一个 Angular 指令来说，宿主元素就是被添加了该指令的元素。举个例子：

```html
<code myHighlightDirective> console.log("Hello, World"); </code>
```

那么这里 `myHighlightDirective` 指令的宿主元素就会是该 `<code>` 元素。

这就是本文的全部内容，下次再见！:wave:
