---
title: let
date: 2018-08-10 18:55:15
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
### let声明的变量在代码块内有效
{% codeblock lang:javascript %}
var a [];
for (let i = 0; i < 10; i++) {
    a[i] = function() {
        console.log(i);
    };
}
a[6](); //6
{% endcodeblock %}
### 不存在变量提升
var命令的变量可以在声明之前使用，值为undefined
### 暂时性死区
在代码块内，使用let命令声明变量之前，该变量都是不可用的。TDZ(temporal dead zone)
{% codeblock lang:javascript %}
var temp = 123;
if (true) {
    temp = 'abc';   //ReferenceError
    let temp;
}
{% endcodeblock %}
有些死区是不易发现的
{% codeblock lang:javascript %}
function bar(x = y, y = 2) {    //y is not defined
    return [x, y];
}
let x = x;  //ReferenceError: x is not defined
{% endcodeblock %}
### 不允许重复声明
{% codeblock lang:javascript %}
function foo() {
    var a;
    let a;
}
{% endcodeblock %}