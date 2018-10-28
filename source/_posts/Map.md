---
title: Map
date: 2018-08-20 12:09:15
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
ES6提供了Map数据结构，它类似对象，也是键值对的集合，但是‘键’的范围不限于字符串，各种类型的值（包括对象）都可以当做键。Map结构是一种更完善的Hash结构实现。如果需要‘键值对’的数据结构，Map比Object更合适。
{% codeblock lang:javascript %}
let m = new Map();
const o = { p : 'hello' };
m.set(o, 'content');
m.get(o);   //'content'
{% endcodeblock %}
Map也可以接受一个数组作为参数，该数组的成员是一个表示键值对的数组：
{% codeblock lang:javascript %}
const map = new Map([
    ['name', '张三'],
    ['title', 'test']
]);

map.size;           //2
map.has('name');    //true
map.get('name');    //'张三'
{% endcodeblock %}
Map构造函数接受数组作为参数，实际上执行的是下面的算法：
{% codeblock lang:javascript %}
const item = [
    ['name', '张三'],
    ['title', 'test']
];
const map = new Map();
item.forEach(
    ([key, valule]) => map.set(key, value)
);
{% endcodeblock %}
### 同名问题
Map的键实际上是和内存地址绑定的，只要内存地址不一样，就视为两个键。如果Map的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map就视其为一个键，包括-0和0。另外，虽然NaN不严格等于自身，但Map将其视为同一个键。
### 实例的属性和操作方法
* size属性
size属性返回Map结构的成员总数。
* set(key, value)
set方法设置key所对应的键值，返回整个Map结构。如果key已经有值，则键值更新，否则新生成键值。
* get(key)
get方法读取key对应的键值，如果找不到key，返回undefined。
* has(key)
has方法返回一个布尔值，表示某个键是否在Map数据结构中。
* delete(key)
delete方法删除某个键，返回true。如果删除失败，返回false。
* clear()
clear方法清除所有成员，没有返回值。
### 遍历方法
* keys()：返回键名的遍历器
* values()：返回键值的遍历器
* entries()：返回所有成员的遍历器
* forEach()：遍历Map的所有成员

**Map的遍历顺序就是插入顺序**
### Map与其他数据结构相互转换
* Map转为数组
{% codeblock lang:javascript %}
const myMap = new Map().set(true, 1).set(false, 2).set({ foo: 1 }, ['abc']);
[...myMap]; //[[true, 1], [false, 2], [{foo:1}, ['abc']]]
{% endcodeblock %}
* 数组转为Map
{% codeblock lang:javascript %}
new Map([
    [true, 1],
    [{ foo : 3 }, ['abc']]
])
{% endcodeblock %}
* Map转为对象
如果Map的所有键都是字符串，则可以转为对象。
{% codeblock lang:javascript %}
function strMapToObj(strMap) {
    let obj = Object.create(null);
    for (let [k, v] of strMap) {
        obj[k] = v;
    }
    return obj;
}
{% endcodeblock %}
* 对象转为Map
{% codeblock lang:javascript %}
function objToStrMap(obj) {
    let strMap = new Map();
    for (let k of Object.keys(obj)) {
        strMap.set(k. obj[k]);
    }
    return strMap;
}
{% endcodeblock %}
* Map转为JSON
1. Map的键名都是字符串：
{% codeblock lang:javascript %}
function strMapToJson(strMap) {
    return JSON.stringify(strMapToObj(strMap));
}
{% endcodeblock %}
2. 键名有非字符串
{% codeblock lang:javascript %}
function mapToArrayJson(map) {
    return JSON.stringify([...map]);
}
{% endcodeblock %}

* JSON转为Map
1. 正常情况下所有键名都是字符串
{% codeblock lang:javascript %}
function jsonToStrMap(jsonStr) {
    return objToStrMap(JSON.parse(jsonStr));
}
{% endcodeblock %}
2. JSON就是一个数组的情况
{% codeblock lang:javascript %}
function jsonToMap(jsonStr) {
    return new Map(JSON.parse(jsonStr));
}
{% endcodeblock %}
### WeakMap
* 只接受对象作为键名（null除外）
* WeakMap中的对象都是弱引用，如果其他对象都不再引用该对象，那么GC会自动回收该对象所占的内存，不考虑该对象是否在WeakMap中
* 没有size属性，没有clear方法

**WeakMap的专用场景就是它的键所对应的对象可能会在将来消失的场景，有助于防止内存泄露**