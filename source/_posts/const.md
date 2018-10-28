---
title: const
date: 2018-08-10 19:07:51
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
### 一旦声明，值不可变
{% codeblock lang:javascript %}
const PI = 3.14;
PI = 3; //TypeError: Assignment to constant variable.
{% endcodeblock %}
只声明不赋值也会报错。
{% codeblock lang:javascript %}
const foo;  //SyntaxError: Missing initializer in const declaration
{% endcodeblock %}
### 实质为变量指向的内存地址不可变动
const只能保证这个指针是固定的，不能控制数据结构的变化。
{% codeblock lang:javascript %}
const foo = {};
foo.prop = 123; //Success
foo = {};       //TypeError: 'foo' is read-only
{% endcodeblock %}
### 对象冻结方法
使用Object.freeze函数，冻结对象
{% codeblock lang:javascript %}
const foo = Object.freeze({});
{% endcodeblock %}
冻结属性的函数
{% codeblock lang:javascript %}
var constantize = (obj) => {
    Object.freeze(obj);
    Object.keys(obj).forEach((key, i) => {
        if (typeof obj[key] === 'object') {
            constantize(obj[key]);
        }
    });
};
{% endcodeblock %}