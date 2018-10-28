---
title: Set
date: 2018-08-18 11:00:05
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
### 基本用法
Set类似于数组，但是成员的值都是唯一的，没有重复。Set本身是一个构造函数。
{% codeblock lang:javascript %}
const set = new Set();
[1, 2, 3, 4].forEach(x => set.add(x));
for (let value of set) {
    console.log(value); //1, 2, 3, 4
}

const set = new Set([1, 2, 3, 4]);
[...set];   //[1, 2, 3, 4]

//数组除重
[...new Set(array)]
{% endcodeblock %}
在Set内部，NaN是相等的，两个对象总是不相等的。
{% codeblock lang:javascript %}
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a).add(b);
set.size;   //1

set.add({}).add({});
set.size;   //3
{% endcodeblock %}
### Set实例的属性和方法
Set结构的实例有以下属性：
* Set.prototype.constructor：构造函数，默认是Set函数
* Set.prototype.size：返回Set成员的总数

Set实例的方法：
* add(value)：添加某个值，返回Set结构本身
* delete(value)：删除某个值，返回布尔值表示是否删除成功
* has(value)：返回一个布尔值，表示参数是否为Set的成员
* clear()：清除所有成员，**没有返回值**

Array.from方法可以将Set转为数组：
{% codeblock lang:javascript %}
const item = new Set([1, 2, 3, 4]);
const array = Array.from(item);

//去重
function dedupe(array) {
    return Array.from(new Set(array));
}
{% endcodeblock %}
### 遍历操作
* keys()：返回键名的遍历器
* values()：返回键值的遍历器
* entries()：返回键值对的遍历器
* forEach()：使用回调函数遍历每个成员

**Set的遍历顺序就是插入顺序**
由于Set结构没有键名，所以values和keys方法的行为完全一致。
### WeakSet
* 成员只能是对象，而不能是其他类型的值
* WeakSet中的对象都是弱引用，如果其他对象都不再引用该对象，那么GC会自动回收该对象所占的内存，不考虑该对象是否在WeakSet中
* 没有size属性，不可遍历