---
title: JSON新方法
date: 2018-08-15 11:42:17
tags:
- '学习'
- '整理'
categories:
- 'HTML5'
---
### eval()
eval可以解析任何字符串变成JS：
{% codeblock lang:javascript %}
var str = "function testFunction() {console.log('test');}";
eval(str);
testFunction(); //'test'
{% endcodeblock %}
### JSON.parse()
JSON.parse只能解析JSON形式的字符串变成JS，安全性比eval高一些。
**字符串中的属性要严格加上引号**
{% codeblock lang:javascript %}
let str = '{ "name" : "hello" }';
let json = JSON.parse(str);
json.name; //'hello'
{% endcodeblock %}
### JSON.stringify()
JSON.stringify将JSON转换成字符串：
{% codeblock lang:javascript %}
let json = {
    name : 'hello'
};
let str = JSON.stringify(json);
str;    //{"name":"hello"}
{% endcodeblock %}
### 复制对象出现的问题
由于=赋值，会有引用的问题，新对象属性改变可能会影响到源对象：
{% codeblock lang:javascript %}
let a = {
    name : 'hello'
};
let b = a;
b.name = 'hi';
a.name; //'hi'
{% endcodeblock %}
可以用JSON的新方法解决：
{% codeblock lang:javascript %}
let a = {
    name : 'hello'
};
let b = JSON.parse(JSON.stringify(a));
b.name = 'ni';
a.name; //'hello'
{% endcodeblock %}
### JS历史管理
触发历史管理的方法：
* 通过跳转页面
* hash
* pushState