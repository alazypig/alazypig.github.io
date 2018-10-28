---
title: Symbol
date: 2018-08-17 15:05:18
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
ES6引入了一种新的原始数据类型Symbol，表示独一无二的值。
Symbol值通过Symbol函数生成，也就是说，对象的属性名现在可以有两种类型：一种是字符串，另一种就是Symbol类型。只要属性名属于Symbol类型，就是独一无二的，可以保证不会与其他属性名冲突。
{% codeblock lang:javascript %}
let s = Symbol();
typeof s;   //symbol
{% endcodeblock %}
Symbol函数可以接受一个字符串作为参数，表示对Symbol实例的描述，主要是为了在console显示。
{% codeblock lang:javascript %}
let s = Symbol('str');
s.toString();   //'Symbol(str)'
{% endcodeblock %}
**如果Symbol的参数是一个对象，就会调用该对象的toString方法，再生成Symbol值**
{% codeblock lang:javascript %}
const obj = {
    toString() {
        return 'test';
    }
};
const s = Symbol(obj);
s.toString();   //'Symbol(test)'
{% endcodeblock %}
**Symbol函数只表示对当前Symbol值的描述，因此相同的Symbol函数的返回值是不相等的**
{% codeblock lang:javascript %}
let s1 = Symbol();
let s2 = Symbol();
s1 === s2;  //false
{% endcodeblock %}
### 作为属性名的Symbol
Symbol值可以作为标识符用于对象的属性名，保证不会出现同名的属性，还能防止某一个键被不小心覆盖。
{% codeblock lang:javascript %}
let symbol = Symbol();

//写法一
let a = {};
a[symbol] = 'test';

//写法二
let a = {
    [symbol]: 'test'
}

//写法三
let a = {};
Object.defineProperty(a, symbol, { value : 'test' });
{% endcodeblock %}
实例：消除代码中的字符串
{% codeblock lang:javascript %}
function getArea(shape, options) {
    let area = 0;
    switch (shape) {
        case 'Triangle':
            area = 0.5 * options.width * options.height;
            break;
    }
    return area;
}
//转变后
const shaptType = {
    triangle: Symbol();
};
function getArea(shape, options) {
    let area = 0;
    switch (shape) {
        case shaptType.triangle:
            area = 0.5 * options.width * options.height;
            break;
    }
    return area;
}
{% endcodeblock %}
### 属性名的遍历
Symbol属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyName()返回。有一个Object.getOwnPropertySymbols方法可以获取指定对象的所有Symbol属性。
{% codeblock lang:javascript %}
let obj = {};
let a = Symbol('hello');
let b = Symbol('world');
obj[a] = 'a';
obj[b] = 'b';
Object.getOwnPropertySymbols(obj);  //[Symbol(hello), Symbol(world)]
{% endcodeblock %}
### Symbol.for()、Symbol.keyFor()
Symbol.for接受一个字符串作为参数，然后搜索有没有以该参数作为名称的Symbol值，如果有，就返回这个Symbol值，否则就新建一个以该字符串为名称的Symbol值。
{% codeblock lang:javascript %}
let str1 = Symbol.for('test');
let str2 = Symbol.for('test');
str1 === str2;  //true
{% endcodeblock %}
Symbol.keyFor方法返回一个已登记的Symbol类型值的key：
{% codeblock lang:javascript %}
let s1 = Symbol.for('foo');
Symbol.keyFor(s1);  //'foo'

let s2 = Symbol('foo');
Symbol.keyFor(s2);  //undefined
{% endcodeblock %}
**Symbol.for为Symbol登记的名字是全局环境的，可以在不同的iframe或serviceWorker中取到同一个值**
### 内置的Symbol值
* Symbol.hasInstance
foo instanceof Foo实际在内部调用的是Foo\[Symbol.hasInstance\](foo)：
{% codeblock lang:javascript %}
class MyClass {
    [Symbol.hasInstance](foo) {
        return foo instanceof Array;
    }
}
[1, 2, 3] instanceof new MyClass(); //true

class Even {
    static [Symbol.hasInstance](obj) {
        return Number(obj) % 2 === 0;
    }
}
1 instanceof Even;  //false
2 instanceof Even;  //true
{% endcodeblock %}
* Symbol.isConcatSpreadable
等于一个布尔值，表示该对象使用Array.prototype.concat()时是否可以展开：
{% codeblock lang:javascript %}
let arr1 = ['a', 'b'];
[1, 2].concat(arr1, 'c');   //[1, 2, 'a', 'b', 'c']
arr1[Symbol.isConcatSpreadable];    //undefined

let arr2 = ['a', 'b'];
arr2[Symbol.isConcatSpreadable] = false;
[1, 2].concat(arr2, 'c');   //[1, 2, ['a', 'b'], 'c']
{% endcodeblock %}
类似数组的对象也可以展开，默认值为false，必须手动打开：
{% codeblock lang:javascript %}
let obj = { length : 2, 0 : 'c', 1 : 'd' };
['a', 'b'].concat(obj); //['a', 'b', obj]
obj[Symbol.isConcatSpreadable] = true;
['a', 'b'].concat(obj); //['a', 'b', 'c', 'd']
{% endcodeblock %}
对一个类而言，Symbol.isConcatSprealable属性必须写成实例属性：
{% codeblock lang:javascript %}
class myClass extends Array {
    constructor(args) {
        super(args);
        this[Symbol.isConcatSpreadable] = true;
    }
}
{% endcodeblock %}
* Symbol.species
对象的Symbol.species属性指向当前对象的构造函数，创造实例时会默认调用这个方法。
{% codeblock lang:javascript %}
class myClass extends Array {
    static get [Symbol.species]() { return Array; }
}
let a = new myClass(1, 2, 3);
let mapped = a.map(x => x * x);
a instanceof Array;         //true
mapped instanceof Array;    //true
a instanceof myClass;       //true
mapped instanceof myClass;  //false

//默认值等同于下面的写法
static get [Symbol.species]() { return this; }
{% endcodeblock %}
* Symbol.match
对象的Symbol.match属性指向一个函数，当执行str.match(myObject)时，如果该属性存在，会调用它的返回值：
{% codeblock lang:javascript %}
String.prototype.match(regexp);
//等同于
regexp[Symbol.match](this);

class myMatcher {
    [Symbol.match](string) {
        return 'hello world'.indexOf(string);
    }
}
e.match(new myMatcher());   //1
{% endcodeblock %}
* Symbol.replace
对象的Symbol.replace属性指向一个方法，当对象被String.prototype.replace方法调用时会返回该方法的返回值：
{% codeblock lang:javascript %}
String.prototype.replace(searchValue, replaceValue);
//等同于
searchValue[Symbol.replace](this, replaceValue);
{% endcodeblock %}
**Symbol.replace方法会收到两个参数，一个是replace方法正在作用的对象，第二个是替换后的值：**
{% codeblock lang:javascript %}
const x = {};
x[Symbol.replace] = (...s) => console.log(s);
'hello'.replace(x, 'world');    //['hello', 'world']
{% endcodeblock %}
* Symbol.search
对象的Symbol.search属性指向一个方法，当对象被String.prototype.search方法调用时会返回该方法的返回值：
{% codeblock lang:javascript %}
String.prototype.search(regexp);
//等同于
regexp[Symbol.search](this);

class mySearch {
    constructor(value) {
        this.value = value;
    }
    [Symbol.search](string) {
        return string.indexOf(this.value);
    }
}
'foobar'.search(new mySearch('foo'));   //3
{% endcodeblock %}
* Symbol.split
对象的Symbol.split属性指向一个方法，当对象被String.prototype.split方法调用时会返回该方法的返回值：
{% codeblock lang:javascript %}
String.prototype.split(separator, limit);
//等同于
separator[Symbol.split](this, limit);
{% endcodeblock %}
重定义split方法的行为：
{% codeblock lang:javascript %}
class mySplit {
    constructor(value) {
        this.value = value;
    }
    [Symbol.split](string) {
        let index = string.indexOf(this.value);
        if (index === -1) return string;
        return [
            string.substr(0, index),
            string.substr(index + this.value.length);
        ];
    }
}
'foobar'.split(new mySplit('foo')); //['', 'bar']
'foobar'.split(new mySplit('bar')); //['foo', '']
'foobar'.split(new mySplit('1'));   //'foobar'
{% endcodeblock %}
* Symbol.iterator
对象的Symbol.iterator属性指向该对象的默认遍历器方法：
{% codeblock lang:javascript %}
let myIterable = {};
myIterable[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
    yield 3;
}
[...myIterable];    //[1, 2, 3]

class Collection {
    *[Symbol.iterator]() {
        let i = 0;
        while (this[i] !== undefined) {
            yield this[i];
            i++;
        }
    }
}
let myCollection = new Collection();
myCollection[0] = 1;
myCollection[1] = 2;

for (let value of myCollection) {
    console.log(value); //1, 2
}
{% endcodeblock %}
* Symbol.toPrimitive
* Symbol.toStringTag
* Symbol.unscopables