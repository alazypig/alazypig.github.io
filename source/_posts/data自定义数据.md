---
title: data自定义数据
date: 2018-08-15 12:17:02
tags:
- '学习'
- '整理'
categories:
- 'HTML5'
---
data自定义数据在query、mobile常用。
{% codeblock lang:html %}
<div id="div1" data-test="hello" data-test-last="world"></div>

<script>
    let oDiv = document.getElementById('div1');
    oDiv.dataset.test;      //'hello'
    oDiv.dataset.testLast;  //'world'
</script>
{% endcodeblock %}