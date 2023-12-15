---
layout: post
author: Dragroo
title: MDN测试答案（持续更新）
tag: 前端
category: 技术
---
## CSS
### 1. [技能测试：表格](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Building_blocks/Tables_tasks '技能测试：表格')
答案：

```css
table {
 table-layout: fixed;
 border-collapse: collapse;
}
th,td{
vertical-align:top;
}
thead th:nth-child(2),thead th:nth-child(3),tfoot th{
 text-align: right;
}
tbody td:nth-child(2),tbody td:nth-child(3){
 text-align:right;
}
thead th:nth-child(1),thead th:nth-child(4){
 text-align:left;
}
tbody th,tbody td:nth-child(4){
 text-align:left;
}
tfoot tr{
 border-top:1px solid black;
 border-bottom:1px solid black;
}
caption{
 border-bottom:1px solid black;
}
tbody tr:nth-child(2n-1){
 background-color:rgb(238, 238, 238);
}
```
这里需要注意的：
1. 要为table的`border-collapse`设置`collapse`值，否则每个单元格的边框都是单独分开的，没办法给某一行设置边框

