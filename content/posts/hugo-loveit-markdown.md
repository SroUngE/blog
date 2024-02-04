---
title: "个人博客(三): Hugo Loveit 自定义Markdown文章样式"           # 文章标题
description : "简单地自定义Loveit主题的Markdown文章样式。"    # 文章描述信息
date: 2023-08-26            # 文章编写日期
tags : [                    # 文章所属标签
    "Hugo",
    "Loveit",
    "Blog"
]
featuredImage: "https://s2.loli.net/2023/05/14/vWZJg25R1CVkXc6.png"
featuredImagePreview: "https://s2.loli.net/2023/05/14/vWZJg25R1CVkXc6.png"
draft: false
---
<!--more-->


> Loveit 自带的markdown 渲染看起来比较乱，原因一是色彩多，二是各个块之间的间距过窄。 这里我会把markdown 尽量改成比较github markdown css的风格，但不会做过多的改动。

## 1. 修改代码样式
Loveit自带的代码引用颜色是红色，看起来非常刺眼，我想要修改为浅黑色。

复制Loveit 的 `blog\themes\Loveit\assets\css\_variable.css` 文件 到自己的css 目录下 `blog\=\assets\css\_variable.css`，修改以下变量为：
```scss
// Color of the code
$code-color: #1F2328 !default; // change code color as light dark

// Color of the code background
$code-background-color: #f5f5f5 !default; // change code bg color as light grey
```
在代码块字体过小，并且中文显示也是不是很好看，这里选用github markdown css 里的font family和size：
```scss
// change code font family to support Chinese
$code-font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji" !default;
$code-font-size: 1rem !default; // turn up font size
```

## 2. 修改各个section的上下间距
首先修改 h1 - h6 ，p的margin和padding，需要修改的代码在`blog\assets\css\_page\_single.scss`，代码如下：
```scss
.single {
  .content {
    h2,
    h3,
    h4,
    h5,
    h6 {
      margin-top: 2rem; // add margin top
      margin-bottom: 1rem; // add margin bottom
      padding-bottom: .3em; // add padding bottom   
    }
    p {
      margin: .5rem 0;
      margin-bottom: 16px; // add margin bottom
    }
  }
}
```

接着要修改`blog\assets\css\_partial\_single\_code.scss`文件来给代码块加边距：
```scss
code {
  padding: .2rem .4rem; // add left and right padding
  border-radius: 6px; // add border radius
  .highlight {
    margin: 1rem 0; // turn up margin top and bottom    
  }
}
```
