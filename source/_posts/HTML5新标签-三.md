---
title: HTML5新标签(三)
date: 2018-08-14 21:30:10
tags:
- '学习'
- '整理'
categories:
- 'HTML5'
---
{% codeblock lang:html %}
<form action="">
    <!--placeholder: 输入框提示信息-->
    <!--autocomplete: 自动保存用户输入过的值，默认为on-->
    <!--pattern: 正则验证-->
    <input type="text" placeholder="请输入6-8个数字" pattern="\d{6,8}" name="user" autocomplete="off" id="" />
    <!--formaction: 定义提交地址-->
    <input type="submit" value="提交" formaction="http://www.baidu.com" />
</form>

<form action="">
    <!--autofocus: 自动焦点-->
    <!--required: 必填项-->
    <input type="text" name="username" id="" autofocus required />
    <input type="password" name="" id="" required />
</form>

<!--datalist选项列表，与input元素配合使用，定义input的可能值-->
<input type="text" list="varList" />
<datalist id="varList">
    <option value="javascript">javascript</option>
    <option value="html">html</option>
    <option value="css">css</option>
</datalist>
{% endcodeblock %}
JS延迟加载
{% codeblock lang:html %}
<!--会在window.onload之前加载-->
<script src="a.js" defer="defer"></script>
<script src="a.js" defer="defer"></script>
<script src="a.js" defer="defer"></script>

<!--异步加载-->
<script src="a.js" async="async"></script>
<script src="b.js" async="async"></script>
<script src="c.js" async="async"></script>
{% endcodeblock %}