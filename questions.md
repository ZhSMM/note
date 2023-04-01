问题列表索引：

1. [markdown实现页内跳转](#1)
2. <a href="#1">跳转到标题</a>



<h4 id="1">markdown实现页内跳转</h4>

markdown 实现页内跳转：

1. 通过HTML标签定义一个锚点，如`<span id="1">锚点</span>`
2. 使用markdown的链接语法：`[点击跳转](#jump)`

```xml
<!-- 定义h4标题锚点 -->
<h4 id="1">标题</h4>

<!-- 跳转 -->
<a href="#1">跳转到标题</a>
```



