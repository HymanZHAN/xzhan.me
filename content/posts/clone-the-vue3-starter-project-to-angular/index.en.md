---
title: "Cloned the Vue 3 starter project in Angular"
subtitle: "How do they compare?"
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

**TL;DR;** The code can be found [here](https://github.com/HymanZHAN/ng-starter-demo).

**Update**: The live demo lives [here](https://ng-starter-demo.netlify.app/).

As indicated by previous blogs, I used to use Vue 2 at work. I haven't been able to do that as I have moved and switched jobs. Now I am dealing with a totally different beast ([SAPUI5](https://ui5.sap.com/) :upside_down_face:). As a close follower of the tech trend (maybe a bit too close :sweat_smile:), I have been long aware of Vite, the new cool kid in town. It"s fast, with good DX and much more, so I am tempted to try it out with a familiar framework, so that"s what I did with the new Vue 3 starter project.

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
{{< / echarts >}}

Notice both apps include a router, and initial payloads don't include the lazy `About` page. My initial impression is two-fold:

- Vue is pretty lightweight. This has always been the case, but a one-to-one comparison makes it much more straightforward.
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
