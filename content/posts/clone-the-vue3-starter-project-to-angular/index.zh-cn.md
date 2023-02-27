---
title: "用Angular克隆了一个Vue3的起始模板项目"
subtitle: "对比一些数据"
date: 2023-02-25T21:13:18+08:00
draft: false
tags: ["angular", "vue"]
categories: ["frontend"]
toc: true
featuredImage: ""
featuredImagePreview: ""
summary: "The Vue 3 + Vite starter project looks nice and clean, so I built an Angular clone of it. How do they compare?"
toc: true
autoCollapseToc: true
---

**太长不看** 代码仓库在[这里](https://github.com/HymanZHAN/ng-starter-demo).

**更新**: 线上项目运行在[这里](https://ng-starter-demo.netlify.app/).

以前博客也提到过，之前的工作中前端开发用的是 Vue 2。但是最近换工作了换到了 SAP，工作上只能用 SAP 自研的 SAPUI5（着实难用:upside_down_face:）。闲暇时光里总还是按捺不住想尝鲜，对 Vite 感兴趣也不是一天两天了，于是打算周末试试看 Vue3 的起始模板，体验体验。新建项目运行起来之后，发现页面还挺清爽，有点内容，排版也听好看，遂开始手痒想着造一个 Angular 同款，看看对比起来如何。于是便有了开头的项目和这篇没什么营养的文章。

{{< admonition warning "不要吵架" false >}}
这里没有踩一捧一的意图，仅仅是纯粹的数字对比。两个我都用两个我都爱，它们都是我的翅膀（大雾）。
{{< /admonition >}}

## 打包大小

下面是一个打包后JS代码大小的对比图：

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

- Vue 还是相当轻量级的。 This has always been the case, but a one-to-one comparison makes it much more straightforward.
- After compression, the difference in downloading time may be less outstanding. Though I still wish one day Angular will hit the 200kB mark, and after that the 100kB one, for this starter project. :smile:

You may ask: But the browser needs to parse and execute those JavaScript, right? Or do they? :thinking:

## Lighthouse

To find out the information on unused JavaScript, we need to dig deeper into the Lighthouse result (which can be found in the repo mentioned on top). You get a treemap with information on unused JavaScript by clicking the "View Treemap" button on the Lighthouse result page.

{{< image src="treemap-angular-min.png" caption="Treemap - Angular" width="80%" >}}

{{< image src="treemap-vue3-min.png" caption="Treemap - Vue" width="80%" >}}

So a more comprehensive graph would look like this:

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

So my third impression:

- Angular has about 35% of unused JavaScript while Vue has about 29%. I am not sure where they live, but the router seems a major suspect. Angular has more room to improve here.

And what about the actual scores and numbers? I know, I know, everyone's favorite, so here you go:

{{< image  src="lighthouse-side-by-side.png" caption="Lighthouse Results" width="100%" >}}

Both frameworks reach 100 on performance, which shouldn't be surprising to anyone. Overall, Vue is about 0.3 seconds faster across the metrics.

## Conclusion

This is a straightforward comparison, so there's nothing to conclude, really. I did get reminded of the "host element". If you don't know Angular, a host element is the HTML element created for an Angular component in the DOM. It matches the component's selector. That means instead of the "clean" HTML that Vue outputs:

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

You get this instead:

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

So when migrating styles from the Vue project, some styles would need to go into the [`:host` selector](https://angular.io/guide/component-styles#host).

As a side note, for an Angular directive, the host element is the element that it gets attached to. For example:

```html
<code myHighlightDirective> console.log("Hello, World"); </code>
```

The host element of the `myHighlightDirective` would be this `<code>` element.

That'd be all for this post. See you next time! :wave:
