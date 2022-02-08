---
title: "CSS面试要点复习"
date: 2022-02-07T23:29:43-05:00
draft: false
tags: ["CSS"]
categories: ["frontend"]
toc: true
featuredImage: ""
featuredImagePreview: ""
summary: "本篇为近期准备前端面试的 CSS 部分个人总结，供个人复习使用。"
toc: true
autoCollapseToc: true
---

本篇为近期准备前端面试的 CSS 部分个人总结，供个人复习使用。

## HTML

### 为什么要 HTML 语义化（semantic HTML）？

- 代码易读性增强，搜索代码更快捷
- 方便搜索引擎对网站排序进行优化（SEO）
- 屏幕阅读器能够更方便地“浏览”网页，对视障人士更为友好

### 块级元素 vs 行内元素

- 块状：占据其父元素（容器）的整个水平空间，垂直空间等于其内容高度，有`display: block / table`，有`div`、`h1/h2/h3`、`table`、`form`、`header`、`p`、`pre`、`ol/ul`等等
- 行内：不以新的一行开始，一般只包括数据，只占据对应标签的边框https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries
  所包含的空间，有`button`、`label`、`select`、`a`、`img`、`code`、`span`、`textarea`等等

## CSS

### 布局

#### 盒模型宽度计算

- offsetWidth = 内容 + 内边距 + 边框，可以理解为渲染出来后“看得见”的部分实际占据的宽度，所以不包括外边距
- 因为 CSS 中定义的`width`、`height`只针对内容，若想 DOM 节点实际的`offsetWidth`和 CSS 中定义的 width 相等的话，需要定义`box-sizing: border-box`

#### margin 纵向重叠

- 相邻元素的`margin-top`、`margin-bottom`会有重叠
- 空白元素（`p`、`div`等）的`margin-top`、`margin-bottom`也不会生效

#### margin 为负时

- `margin-top`、`margin-left`负值，元素自身向上、向左移动
- `margin-right`为负，右侧元素向左移动，自身不受影响
- `margin-bottom`为负，下方元素向上移动，自身不受影响

#### BFC 理解与应用

- 什么是 BFC： 一块独立的渲染区域，内部元素会被包裹起来，其渲染不受外界影响，也可用于不希望这块区域内部的渲染也影响到整个页面的布局的情况
- 应用：
  - 不影响整体页面布局，如底部反馈按钮设置为`position: absolute; bottom: 50px; right: 50px`
  - 清除浮动脱离文档流导致的背景塌陷等副作用，可以把 float 元素的父元素设置为`overflow: auto/hidden/scroll`或者`display: flex/inline-box`创建 BFC
- 清除父子元素外边距塌陷的问题，给子元素添加`display: flex/inline-box`可以把子元素包裹起来，这时子元素的外边距就是相对父元素而不是父元素外部了
- 常见方式：
  - `position`: absolute/fixed
  - `float`: not none
  - `overflow`: not visible
  - `display`: flex/inline-block
  - root: `<html>`

#### float 布局

- 圣杯布局

  - 设置 container 左右的内边距
  - 设置左、右元素的 width 为对应的内边距，设置中央元素宽度为 100%
  - 全部元素`float: left`
  - HTML 如下：
    ```html
    <div id="container">
      <div id="center" class="column">this is center</div>
      <div id="left" class="column">this is left</div>
      <div id="right" class="column">this is right</div>
    </div>
    ```
  - `#left { margin-left: -100% }` 相当于左移一个`#center`元素的宽度，刚好移动到了中央元素左边
  - `#right { margin-right: -100% }` 相当于把右元素后的元素往左移动，但是因为`#right`右边没有元素了，所以相当于把自身往右移动 100% 父元素的宽度，也就是`#center`的宽度

- 双飞翼布局

  - HTML 布局如下：
    ```html
    <body>
      <div id="main" class="col">
        <div id="main-wrap">this is main</div>
      </div>
      <div id="left" class="col">this is left</div>
      <div id="right" class="col">this is right</div>
    </body>
    ```
  - 所有`col`都`float: left`
  - `#main`宽度为 100%，`main-wrap`设置外边距，对应左右元素的宽度
  - `#left { margin-left: -100% }`
  - `#right { margin-left: <自身宽度> }`

- 手写 clear fix
  ```css
  .clearfix:after {
    content: "";
    display: table;
    clear: both;
  }
  ```

#### flex 布局

- https://css-tricks.com/snippets/css/a-guide-to-flexbox/
- https://tympanus.net/codrops/css_reference/flexbox/

### 定位

> 仔细研读 https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Positioning

#### absolute 和 relative 定位

- `relative`根据自身定位
- `absolute`一层层往上找父元素，直到找到最近的一个定位元素为止
  - 定位元素包含`position: absolute/fixed/relative/static`和`body`等

#### 居中对齐

- 水平居中
  - 对块级元素当中的**内容**进行水平居中：`text-align: center`
  - 让块级元素**本身**在父元素中居中：`margin: auto`
  - 若`display: absolute`，可以`left: 50%; margin-left: -<自己一半的宽度>`
- 垂直居中

  - 行内元素：`line-height`=`height`
  - 若`display: absolute`，可以`top: 50%; margin-top: -<自己一半的高度>`（显而易见，自身高度必须已知）
  - 若`display: absolute`，还可以`top: 50%; left: 50%; transform: translate(-50%, -50%)`
  - （终极）方案：
    ```css
     {
      position: absolute;
      top: 0;
      left: 0;
      bottom: 0;
      right: 0;
      margin: auto;
    }
    ```
    其中，`top:0; bottom:0; margin: auto`可以做到垂直方向居中，`left:0; right:0; margin: auto`可以做到水平方向居中。

### 图文样式

#### line-height 如何继承

> 仔细阅读 https://developer.mozilla.org/en-US/docs/Web/CSS/line-height

- `line-height`最好使用数字或像素，因为使用`em`和百分比都会出现继承问题
- 使用像素和数字，计算行高时会用自身字体大小
- 使用`em`和百分比，计算行高会用父元素字体大小

### 响应式布局

#### rem

- `em`相对父元素的字体大小
- `rem`相对根元素的字体大小（`r` for root）

#### media-query

> https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries
