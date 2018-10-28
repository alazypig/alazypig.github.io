---
title: Math的扩展
date: 2018-08-11 15:56:44
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
mathjax: true
---
### Math.trunc()
Math.trunc用于除去一个数的小数部分。
{% codeblock lang:javascript %}
Math.trunc(111.22);     //111
Math.trunc('123.456');  //123
//对空和无法截取整数的值，返回NaN
Math.trunc(NaN);        //NaN

//代码模拟
Math.trunc = Math.trunc || function (x) {
    return x < 0 ? Math.ceil(x) : Math.floor(x);
};
{% endcodeblock %}
### Math.sign()
Math.sign用于判断一个数到底是正数、负数、还是零。
* 参数为正数，返回+1
* 参数为负数，返回-1
* 参数为0，返回0
* 参数为-0，返回-0
* 其他值，返回NaN

{% codeblock lang:javascript %}
//代码模拟
Math.sign = Math.sign || function (x) {
    x = +x; //convert to a number
    if (x === 0 || isNaN(x)) {
        return x;
    }
    return x > 0 ? 1 : -1;
};
{% endcodeblock %}
### Math.cbrt()
Math.cbrt用于计算一个数的立方根，对于非数值，内部也是先使用Number方法转换为数值。
{% codeblock lang:javascript %}
//代码模拟
Math.cbrt = Math.cbrt || function (x) {
    var y = Math.pow(Math.abs(x), 1 / 3);
    return x < 0 ? -y : y;
};
{% endcodeblock %}
### Math.clz32()
Math.clz32返回一个数的32位无符号整数形式有多少个前导0.
{% codeblock lang:javascript %}
Math.clz32(0);      //32
Math.clz32(1);      //31
Math.clz32(0b01000000000000000000000000000000); //1

Math.clz32(1 << 1); //30
{% endcodeblock %}
**对于小数，Math.clz32()只考虑整数部分**
{% codeblock lang:javascript %}
Math.clz32(3.9);    //30
{% endcodeblock %}
### Math.imul()
Math.imul返回两个数以32位带符号整数形式相乘的结果，返回的也是一个32位带符号整数。
**对于很大数的乘法，低位数值往往是不精确的，Math.imul()方法可以正确的返回低位数值。**
### Math.fround()
Math.fround方法返回一个数的单精度浮点数形式。
**对于整数来说，Math.fround()方法的返回结果不会有任何不同，区别主要在于那些无法用64个二进制位精确表示的小数。这时，Math.fround()方法会返回最接近这个小数的单精度浮点数。**
{% codeblock lang:javascript %}
//代码模拟
Math.fround = Math.fround || function (x) {
    return new Float32Array([x])[0];
};
{% endcodeblock %}
### Math.hypot()
Math.hypot方法返回所有参数的平方和的平方根。
{% codeblock lang:javascript %}
Math.hypot(3, 4);   //5
{% endcodeblock %}
### Math.expm1()
Math.expm1(x)返回$e^x-1$， 即Math.exp(x) - 1;
{% codeblock lang:javascript %}
//代码模拟
Math.expm1 = Math.expm1 || function (x) {
    return Math.exp(x) - 1;
};
{% endcodeblock %}
### Math.log1p()
Math.log1p(x)返回$ln(1+x)$，即Math.log(1 + x)。如果x小于-1，则返回NaN。
{% codeblock lang:javascript %}
//代码模拟
Math.log1p = Math.log1p || function (x) {
    return Math.log(1 + x);
}
{% endcodeblock %}
### Math.log10()
Math.log10(x)返回$log_{10}(x)$，如果x小于0，返回NaN。
{% codeblock lang:javascript %}
//代码模拟
Math.log10 = Math.log10 || function (x) {
    return Math.log(x) / Math.LN10;
};
{% endcodeblock %}
### Math.log2()
Math.log2(x)返回$log_2(x)$，如果x小于0，返回NaN。
{% codeblock lang:javascript %}
//代码模拟
Math.log2 = Math.log2 || function (x) {
    return Math.log(x) / Math.LN2;
};
{% endcodeblock %}
### 双曲函数方法
ES6新增了六个双曲函数方法
* Math.sinh(x) 返回 $sinh\ x=\frac{e^x-e^{-x}}{2}$
* Math.cosh(x) 返回 $cosh\ x=\frac{e^x+e^{-x}}{2}$
* Math.tanh(x) 返回 $tanh\ x=\frac{sinh\ x}{cosh\ x}=\frac{e^x-e^{-x}}{e^x+e^{-x}}$
* Math.asinh(x) 返回 $arsinh\ x=ln(x+\sqrt[\ ]{x^2+1})$
* Math.acosh(x) 返回 $arsinh\ x=ln(x+\sqrt[\ ]{x^2-1})$
* Math.atanh(x) 返回 $artanh\ x=\frac{1}{2}ln\frac{1+x}{1-x}$
### Math.sign()
Math.sign用来判断一个数的正负，如果参数是-0，会返回-0。
### 指数运算符**
{% codeblock lang:javascript %}
2 ** 2;     //4
3 ** 3;     //27

let a = 1.5;
a **= 3;    //a = a * a * a;
{% endcodeblock %}
**在V8引擎中，指数运算符与Math.pow的实现不相同，对于特别大的运算结果，两者会有差异**