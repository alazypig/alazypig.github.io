---
title: Number的扩展
date: 2018-08-11 12:37:20
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
mathjax: true
---
### 进制表示
0b(0B)表示二进制，0o(0O)表示八进制
### Number.isFinite()、Number.isNaN()
Number.isFinite用来检查一个数值是否为有限的。
{% codeblock lang:javascript %}
Number.isFinite(NaN);       //false
Number.isFinite(Infinity);  //false
Number.isFinite(-Infinity); //false
Number.isFinite('15');      //false
Number.isFinite('foo');     //false
Number.isFinite(true);      //false
{% endcodeblock %}
ES5部署Number.isFinite()方法：
{% codeblock lang:javascript %}
(function (global) {
    var global_isFinite = global.isFinite;

    Object.defineProperty(Number, 'isFinite', {
        value : function isFinite(value) {
            return typeof value === 'number' && global_isFinite(value);
        },
        configurable : true,
        enumerable : false,
        writable : true
    });
})(this);
{% endcodeblock %}
Number.isNaN用来检查一个值是否为NaN。
{% codeblock lang:javascript %}
Number.isNaN(NaN);              //true
Number.isNaN('15');             //false
Number.isNaN(9 / NaN);          //true
Number.isNaN('true' / 0);       //true
Number.isNaN('true' / 'true');  //true
{% endcodeblock %}
ES5部署Number.isNaN()方法：
{% codeblock lang:javascript %}
(function (global) {
    var global_isNaN = global.isNaN;

    Object.defineProperty(Number, 'isNaN', {
        value : function isNaN(value) {
            return typeof value === 'number' && global_isNaN(value);
        },
        configurable : true,
        enumerable : false,
        writable : true
    });
})(this);
{% endcodeblock %}
### Number.parseInt()、Number.parseFloat()
ES6将全局方法parseInt和parseFloat移植到了Number对象上面，行为完全不变。
### Number.isInteger()
Number.isInteger判断一个值是否为整数。在JavaScript内部，整数和浮点数是相同的储存方法，所以3和3.0视为同一个值。
{% codeblock lang:javascript %}
Number.isInteger('15'); //false
Number.isInteger(true); //false
{% endcodeblock %}
ES5部署Number.isInteger()方法：
{% codeblock lang:javascript %}
(function (global) {
    var floor = Math.floor, isFinite = global.isFinite;

    Object.defineProperty(Number, 'isInteger', {
        value : function isInteger(value) {
            return typeof value === 'number' && isFinite(value) && floor(value) == value;
        },
        configurable : true,
        enumerable : false,
        writable : true
    });
})(this);
{% endcodeblock %}
### Number.EPSILON
ES6在Number对象上面新增一个极小的常量--**Number.EPSILON**。
浮点数计算是不精确的，但是如果这个误差可以小于Number.EPSILON，就可以认为得到了正确的结果。
### Number.isSafeInteger()
JavaScript能够准确表示的整数范围在$-2^{53}$到$2^{53}$之间（不含两个端点），超过这个范围就无法精确表示。
ES6引入了 **Number.MAX_SAFE_INTEGER** 和 **Number.MIN_SAFE_INTEGER** 两个常量来表示这个范围的上下限