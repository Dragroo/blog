---
layout: post
author: Dragroo
title: 技能测试：背景与边框-MDN—CSS教程
tag: 前端 CSS
category: 技术
---
### 问题原文
![问题原文](../images/1212/屏幕截图%202023-12-12%20101536.png)
```html
<div class="box">
  <h2>Backgrounds & Borders</h2>
</div>
```
#### 关于任务2的疑问
除了边框相关的样式应当在`.box`选择器中修改外，其余的文字居中和背景设置应当都在`h2`选择器中完成（右边三个星星的高度和h2的高度是一样的）。
#### 答案如下
```css
.box {
    border: 5px solid lightblue;
    border-top-left-radius: 20px;
    border-bottom-right-radius: 40px;
}
h2{
    background-image: url(star.png), url(star.png);
    background-position: left center, right center;
    background-repeat: no-repeat, repeat-y;
    text-align: center;
}
```