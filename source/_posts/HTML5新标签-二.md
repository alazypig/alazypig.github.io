---
title: HTML5新标签(二)
date: 2018-08-14 21:22:30
tags:
- '学习'
- '整理'
categories:
- 'HTML5'
---
{% codeblock lang:html %}
<!--新的输入控件-->

<!--email: 电子邮箱文本框，当输入的不是邮箱的时候，验证通不过。移动端的键盘会有变化-->
<form action="">
    <input type="email" name="" id="" />
    <input type="submit" value="提交" />
</form>

<!--tel: 电话号码-->
<input type="tel" name="" id="" />
<!--url: 网页的url-->
<!--search: 搜索引擎，chrom下会多个关闭按钮-->
<!--number: 只能输入数字-->
<!--color: 颜色选择器-->
<!--datetime: 显示日期-->
<!--datetime-local: 显示完整日期，不含时区-->
<!--time: 显示时间，不含时区-->
<!--date: 显示日期-->
<!--week: 显示周-->
<!--month: 显示月-->

<!--特定范围内的数值选择器 max、min、step、value-->
<input type="range" step="2" min="0" max="10" value="2" name="" id="" />

{% endcodeblock %}