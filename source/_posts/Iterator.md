---
title: Iterator
date: 2018-08-31 22:15:20
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
### Iterator概念
Iterator是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构，只要部署Iterator接口，就可以完成遍历操作。
Iterator的主要作用：为数据结构提供统一的、简便的访问接口；使得数据结构的成员能够按照某种次序排列；供for...of消费。
遍历过程如下：
1. 创建一个指针对象，指向当前数据结构的起始位置。
2. 第一次调用指针对象的next方法，将指针指向数据结构的第一个成员。
3. 第二次调用next方法，指向第二个成员。
4. 不断调用next方法，直到指针指向数据结构的结束位置。

每次调用next方法都会返回数据结构当前成员的信息，返回一个包含value的done两个属性的对象。value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。
模拟next方法返回值：
{% codeblock lang:javascript %}
var it = makeIterator(['a', 'b']);
it.next();  // { value : 'a', done : false }
it.next();  // { value : 'b', done : false }
it.next();  // { value : undefined, done : true }
function makeIterator(array) {
    var nextIndex = 0;
    return {
        next : function () {
            return nextIndex < array.length ?
            { value : array[nextIndex++], done : false } :
            { value : undefined, done : true };
        }
    };
}
{% endcodeblock %}
遍历器与所遍历的数据结构实际上是分开的，完全可以写出没有对应数据结构的遍历器对象，或者用遍历器对象模拟出数据结构。无限运行的遍历器对象的例子：
{% codeblock lang:javascript %}
var it = idMaker();
it.next().value;    // 0
it.next().value;    // 1
//...
function idMaker() {
    var index = 0;
    return {
        next : function () {
            return { value : index++, done : false };
        }
    };
}
{% endcodeblock %}
### 默认Iterator接口
类部署Iterator接口：
{% codeblock lang:javascript %}
class RangeIterator {
    constructor(start, stop) {
        this.value = start;
        this.stop = stop;
    }
    [Symbol.iterator]() { return this; }
    next() {
        var value = this.value;
        if (value < this.stop) {
            this.value++;
            return { value : value, done : false };
        }
        return { value : undefined, done : true };
    }
}
function range(start, stop) {
    return new RangeIterator(start, stop);
}
for (var value of range(0, 3)) {
    console.log(value); // 0, 1, 2
}
{% endcodeblock %}
实现指针结构：
{% codeblock lang:javascript %}
function Obj(value) {
    this.value = value;
    this.next = null;
}
Obj.prototype[Symbol.iterator] = function () {
    var iterator = { next : next };
    var current = this;
    function next() {
        if (current) {
            var value = current.value;  // 获取当前值
            current = current.next;     // 指向下一个实例
            return { value : value, done : false };
        } else {
            return { done : true };
        }
    }
    return iterator;
}
var one = new Obj(1);
var two = new Obj(2);
var three = new Obj(3);

one.next = two;
two.next = three;

for (var i of one) {
    console.log(i); // 1, 2, 3
}
{% endcodeblock %}
为对象添加Iterator接口：
{% codeblock lang:javascript %}
let obj = {
    data : ['hello', 'world'],
    [Symbol.iterator]() {
        const self = this;
        let index = 0;
        return {
            next() {
                if (index < self.data.length) {
                    return {
                        value : self.data[index++],
                        done : false
                    };
                } else {
                    return { value : undefined, done : true };
                }
            }
        };
    }
};
{% endcodeblock %}
类似数组对象调用数组的Symbol.iterator方法：
{% codeblock lang:javascript %}
let iterable = {
    0 : 'a',
    1 : 'b',
    2 : 'c',
    length : 3,
    [Symbol.iterator] : Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
    console.log(item);  // 'a', 'b', 'c'
}
{% endcodeblock %}
普通对象部署数组的Symbol.iterator方法并无效果。
**如果Symbol.iterator方法对应的不是遍历器生成对象（即会返回一个遍历器对象），解释引擎将会报错。**
### 调用Iterator的场合
* 解构赋值
{% codeblock lang:javascript %}
let set = new Set().add('a').add('b').add('c');
let [x, y] = set;           // x = a, y = b
let [first, ...rest] = set; // first = 'a', rest = ['b', 'c']
{% endcodeblock %}
* 拓展运算符
{% codeblock lang:javascript %}
var str = 'hello';
[...str];           // ['h', 'e', 'l', 'l', 'o']
let arr = ['b', 'c'];
['a', ...arr, 'd']; // ['a', b', 'c', 'd']
{% endcodeblock %}
* yield*
yield*后面跟着是一个可遍历的结构，它会调用该结构的遍历器接口。
{% codeblock lang:javascript %}
let generator = function* () {
    yield 1;
    yield* [2, 3, 4];
    yield 5;
};
var iterator = generator();
iterator.next();    // { value : 1, done : false }
iterator.next();    // { value : 2, done : false }
iterator.next();    // { value : 3, done : false }
iterator.next();    // { value : 4, done : false }
iterator.next();    // { value : 5, done : false }
iterator.next();    // { value : undefined, done : true }
{% endcodeblock %}

### 字符串的Iterator接口
覆盖原生的Symbol.iterator方法达到修改遍历器行为的目的。
{% codeblock lang:javascript %}
var str = new String('hi');
[...str];   // ['h', 'i']
str[Symbol.iterator] = function () {
    return {
        next : function () {
            if (this._first) {
                this._first = false;
                return { value : 'bye', done : false };
            } else {
                return { done : true };
            }
        },
        _first : true
    };
};
[...str];   // ['bye']
str;        //'hi'
{% endcodeblock %}
### Iterator接口与Generator函数
Symbol.iterator方法的最简单实现还是使用Generator函数。
{% codeblock lang:javascript %}
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
    yield 3;
};
[...myIterable];    // [1, 2, 3]
//或下面的写法
let obj = {
    *[Symbol.iterator]() {
        yield 'hello';
        yield 'world';
    }
};
for (let x of obj) {
    console.log(x);
}
{% endcodeblock %}
### return()、throw()
return使用的场合是，如果for...of循环提前退出（error、continue、break)，会调用return方法。如果一个对象在完成遍历以前需要清理或释放资源，就可以部署return方法。
{% codeblock lang:javascript %}
function readLinesSync(file) {
    return {
        next() {
            return { done : false };
        },
        return() {
            file.close();
            return { done : true };
        }
    };
}
for (let line of readLinesSync(fileName)) {
    console.log(line);
    break;
}
{% endcodeblock %}
**return必须返回一个对象，这是Generator规定的**
### 数组
数组原生具备Iterator接口：
{% codeblock lang:javascript %}
const arr = ['a', 'b', 'c'];
const obj = {};
obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);
for (let v of arr) console.log(v);
for (let v of obj) console.log(v);
//完全相同的结果
{% endcodeblock %}
for...of可以代替forEach方法，for...in获取对象的键名，for...of获取对象的键值：
{% codeblock lang:javascript %}
var arr = ['a', 'b', 'c', 'd'];
for (let a in arr) console.log(a);  // 0 1 2 3
for (let a of arr) console.log(a);  // 'a' 'b' 'c' 'd'
{% endcodeblock %}
for...of只返回具有数字索引的属性：
{% codeblock lang:javascript %}
let arr = [3, 4, 5];
arr.foo = 'fun';
for (let i in arr) console.log(i);  // 0 1 2 foo
for (let i of arr) console.log(i);  // 3 4 5
{% endcodeblock %}
### Set和Map结构
Set和Map结构原生具有Iterator接口，可以直接使用for...of。
{% codeblock lang:javascript %}
var engines = new Set(['aaa', 'bbb', 'ccc']);
for (var e of engines) console.log(e);  // aaa bbb ccc

var es = new Map();
es.set('edition', 6);
es.set('committee', 'TC39');
for (var [name, value] of es) console.log(name + ' : ' + value);
// edition : 6
// committee : TC39
{% endcodeblock %}
### 其他方法
* entries()返回一个遍历器对象，用于遍历[键名，键值]组成的数组
* keys()返回一个遍历器对象，用于遍历所有键名
* values()返回一个遍历器对象，用于遍历所有键值

### 类似数组对象
用Array.from()方法转为数组：
{% codeblock lang:javascript %}
let arrayLike = { length : 2, 0 : 'a', 1 : 'b' };
for (let x of Array.from(arrayLike)) console.log(x);    // 'a' 'b'
{% endcodeblock %}
### 普通对象
for...in仍可用于遍历键名，但是for...of不能使用，一种解决方法是使用Object.keys()生成一个键名数组：
{% codeblock lang:javascript %}
for (var key of Object.keys(object)) console.log(key + ':' + object[key]);
{% endcodeblock %}
另一个方法是使用Generator函数将对象重新包装：
{% codeblock lang:javascript %}
function* entries(obj) {
    for (let key of Object.keys(obj)) {
        yield [key, obj[key]];
    }
}for (let [key, value] of entries(obj)) {
    console.log(key + '->' + value);
}
{% endcodeblock %}
for...in的不足：
1. 数组的键名是数字，但是for...in循环是以字符串作为键名，'0'、'1'等
2. for...in循环不仅会遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键
3. 某些情况下，for...in会以任意顺序遍历键名