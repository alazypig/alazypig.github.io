---
title: HTML5新标签(一)
date: 2018-08-14 20:04:00
tags:
- '学习'
- '整理'
categories:
- 'HTML5'
---
{% codeblock lang:html %}
<!--语义化标签-->
<header>页面的头部</header>
<hgroup>
    <h1>Test</h1>
    <h2>test</h2>
</hgroup>
<footer>页面的底部</footer>
<nav>
    <a href="#">导航</a>
    <a href="#">link1</a>
    <a href="#">link2</a>
</nav>
<section>用来划分区域</section>
<article>用来在页面中表示一套结构完整且独立的内容部分（主题）</article>
<aside>和主题相关的附属信息</aside>


<figure>用于对元素进行组合，一般用于图片或视屏</figure>
<figure>
    <img src="" alt="" />
    <figcaption>Test</figcaption>
</figure>

<time>9:00</time>
<p>明天
    <time datatime="2018-02-14">情人节</time>
</p>

<!--用于描述文档或文档某个部分的细节-->
<details open>
    <!--open时默认打开-->
    <summary>test</summary>
    <!--details元素的标题-->
    <p>testjfkdsjkfsjd</p>
</details>

<!--定义一段对话-->
<dialog>
    <dt>老师</dt>
    <dd>2 + 3 ?</dd>
    <dt>学生</dt>
    <dd>5</dd>
</dialog>

<address>定义文章或页面作者的详细联系信息</address>

<mark>需要标记的词或句子</mark>

<!--keygen给表单添加一个公钥-->
<form action="http://www.baidu.com" method="get">
    用户：
    <input type="text" name="usr_name" /> 公钥：
    <keygen name="security" />
    <input type="submit" />
</form>

<!--定义进度条-->
<progress max="100" value="76">
    <span>76</span>%
    <!--为了兼容-->
</progress>
{% endcodeblock %}
