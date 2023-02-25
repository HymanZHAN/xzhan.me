---
title: "Building an Angular Starter Project"
subtitle: "and how does it compare to the Vue one"
date: 2023-02-25T21:13:18+08:00
draft: true
tags: ["angular", "vue"]
categories: ["frontend"]
toc: true
featuredImage: ""
featuredImagePreview: ""
summary: "**TL;DR;** The code can be found here"
toc: true
autoCollapseToc: true
---

TL;DR; The code can be found here.

As indicated by previous blogs, I use Vue 2 at work. I haven"t been able to do that as I have moved and switched jobs. Now I am dealing with a totally different beast ([SAPUI5](https://ui5.sap.com/) :upside_down_face:). As a close follower of the tech trend (maybe a bit too close :)), I have been long aware of Vite, the new cool kid in town. It"s fast, with good DX and much more, so I am tempted to try it out with a familiar framework, so that"s what I did with the new Vue 3 starter project.

I am pleasantly surprised by the new starter app. The default starter page looks nice and clean, with good information and a nice layout. It immediately occurred to me: Can I clone an Angular version of this? For fun and for performance comparison or whatnot? That seems like a good weekend project. So here"s the result.

{{< admonition warning "Not here to FIGHT" false >}}
This is not meant to bash one framework or the other. I love both, and this is a comparison that stems from pure curiosity.
{{< /admonition >}}

## Bundle Size

Here is a chart of the bundle size for the initial page load of both frameworks. 🟥 for Angular and 🟩 for Vue, of course.
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
{{< /echarts >}}

Notice both apps include a router, and the lazy `About` page is not included. My initial impression is two-fold:

- Vue is really lightweight. This has always been the case but a one-to-one comparison makes it much more direct.
- After compression, the difference in downloading time may not be as outstanding. But still, those JavaScript code need to be processed by the browser. Or do they? :thinking:

A more interesting observation can be found by digging deeper into the Lighthouse result. By clicking the "View Treemap" button, you get a treemap with information on unused JavaScript.
