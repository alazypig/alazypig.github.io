---
title: Generator
date: 2018-09-01 14:21:05
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
### 基本用法
{% codeblock lang:javascript %}
function* testGenerator() {
    yield 'hello';
    yield 'world';
    return 'done';
}
let test = testGenerator();
test.next();    // { value : 'hello', done : false }
test.next();    // { value : 'world', done : false }
test.next();    // { value : 'done', done : true }
test.next();    // { value : undefined, done : true }
{% endcodeblock %}
### yield表达式
Generator的next方法运行逻辑如下：
1. 遇到yield语句就暂停执行后面的操作，并将紧跟在yield后的表达式的值作为的对象的value属性值
2. 下一次调用next方法时在继续往下执行，直到遇到下一条yield语句
3. 如果没有遇到新的yield语句，就一直运行到函数结束，知道运行到return为止，并将return语句后面的表达式作为返回对象的value属性值
4. 如果该函数没有return语句，则返回对象的value属性值为undefined

yield是惰性求值的：
{% codeblock lang:javascript %}
function* gen() {
    yield 123 + 456;
}
{% endcodeblock %}
以上的代码只有在next将指针移到这一句时才求值。
Generator函数可以不使用yield语句，此时就变成了一个暂缓执行的函数，只有在调用了next时才执行：
{% codeblock lang:javascript %}
function* f() {
    console.log('run');
};
let generator = f();
setTimeout(() => generator.next(), 3000);
{% endcodeblock %}
展开数组嵌套：
{% codeblock lang:javascript %}
var arr = [[1, 2], 3, [4, [5]]];
var flat = function* (arr) {
    var length = arr.length;
    for (var i = 0; i < length; i++) {
        var item = arr[i];
        if (typeof item !== 'number') {
            yield* flat(item);
        } else {
            yield item;
        }
    }
};
for (var f of flat(arr)) {
    console.log(f); // 1, 2, 3, 4, 5
}
{% endcodeblock %}
### next方法的参数
next可以带有一个参数，该参数会被当做上一条yield语句的返回值，这样就可以在Generator函数运行的不同阶段从外部向内部注入不同的值，从而调整函数的行为：
{% codeblock lang:javascript %}
function* f() {
    for (var i = 0; true; i++) {
        var reset = yield i;
        if (reset) { i = -1; }
    }
}
let generator = f();
generator.next();   // { value : 0, done : false }
generator.next();   // { value : 1, done : false }
generator.next();   // { value : 2, done : false }
generator.next(true);   // { value : 0, done : false }
generator.next();   // { value : 1, done : false }
generator.next();   // { value : 2, done : false }
{% endcodeblock %}
**V8引擎直接忽略第一次使用next时的参数，只有第二次使用next开始的参数才是有效的**
向内部注入值的例子：
{% codeblock lang:javascript %}
function* dataConsumer() {
    console.log('start');
    console.log(`1. ${yield}`);
    console.log(`2. ${yield}`);
    return 'result';
}
let run = dataConsumer();
run.next();         // 'start'
run.next('first');  // 1. first
run.next('haha');   // 2. haha
{% endcodeblock %}
### Generator.prototype.throw()
throw方法可以在函数体外抛出错误，然后在Generator函数体内捕获：
{% codeblock lang:javascript %}
var g = function* () {
    try {
        yield;
    } catch (e) {
        console.log('内部捕获', e);
    }
};
var i = g();
i.next();
try {
    i.throw('a');   // 内部捕获 a
    i.throw('b');   // 外部捕获 b
} catch (e) {
    console.log('外部捕获', e);
}
{% endcodeblock %}
throw方法可以接受一个参数，该参数会被catch语句接收：
{% codeblock lang:javascript %}
var g = function* () {
    try {
        yield;
    } catch (e) {
        console.log(e);
    }
};
var i = g();
i.next();
i.throw(new Error('test'));
// Error: test
{% endcodeblock %}
如果Generator函数内部部署了try...catch代码块，那么遍历器的throw方法抛出的错误不影响下一次遍历，否则遍历直接终止。
**遍历器的throw与throw不同，后者只能被函数体外的catch捕获到**
throw方法被捕获后会附带执行下一条yield表达式，即执行一次next方法：
{% codeblock lang:javascript %}
var gen = function* () {
    try {
        yield console.log('a');
    } catch (e) {}
    yield console.log('b');
    yield console.log('c');
}
var g = gen();
g.next();   // 'a'
g.throw();  // 'b'
g.next();   // 'c'
{% endcodeblock %}
Generator函数体内抛出的错误也能被函数体外的catch捕获：
{% codeblock lang:javascript %}
function* foo() {
    var x = yield 3;
    var y = x.toUpperCase();
    yield y;
}
var it = foo();
it.next();  // { value : 3, done : false }
try {
    it.next(32);
} catch (e) {
    console.log(e); //TypeError
}
{% endcodeblock %}
一旦Generator执行过程中抛出错误，就不会再执行下去。如果此后再调用next，将返回一个value属性等于undefined，done属性等于true的对象。
### Generator.prototype.return()
返回给定的值，并终结Generator函数的遍历：
{% codeblock lang:javascript %}
function* gen() {
    yield 1;
    yield 2;
    yield 3;
}
var g = gen();
g.next();           // { value : 1, done : false }
g.return('foo');    // { value : 'foo', done : true }
g.next();           // { value : undefined, done : true }
{% endcodeblock %}
如果Generator函数内部有try...catch代码块，那么return方法会推迟到finally代码块执行完成再执行。
### yield*
用于在一个Generator函数中执行另一个Generator函数。
{% codeblock lang:javascript %}
function* foo() {
    yield 'a';
    yield 'b';
}

function* bar() {
    yield 'x';
    yield* foo();
    yield 'y';
}
//等同于
function* bar() {
    yield 'x';
    yield 'a';
    yield 'b';
    yield 'y';
}
{% endcodeblock %}
用yield*取出嵌套数组成员：
{% codeblock lang:javascript %}
function* iterTree(tree) {
    if (Array.isArray(tree)) {
        for (let i = 0; i < tree.length; i++) {
            yield* iterTree(tree[i]);
        }
    } else {
        yield tree;
    }
}
const tree = ['a', ['b', 'c'], ['d', 'e']];
for (let x of iterTree(tree)) console.log(x);   // a b c d e
{% endcodeblock %}
遍历完全二叉树：
{% codeblock lang:javascript %}
function Tree(left, label, right) {
    this.left = left;
    this.label = label;
    this.right = right;
}
//中序遍历函数
function* inorder(t) {
    if (t) {
        yield* inorder(t.left);
        yield t.label;
        yield* inorder(t.right);
    }
}
//生成二叉树
function make(array) {
    if (array.length === 1) return new Tree(null, array[0], null);
    return new Tree(make(array[0]), array[1], make(array[2]));
}
let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);
var result = [];
for (let node of inorder(tree)) {
    result.push(node);
}
result; // ['a', 'b', 'c', 'd', 'e', 'f', 'g']
{% endcodeblock %}
### 作为对象属性的Generator函数
{% codeblock lang:javascript %}
let obj = {
    *myGeneratorMethod() {
        //...
    }
}
//等价写法
let obj = {
    myGeneratorMethod : function* () {
        //...
    }
}
{% endcodeblock %}
### Generator函数this
Generator函数总是返回一个遍历器，ES6规定这个遍历器是Generator函数的实例，它也继承了Generator函数的prototype对象上的方法。
{% codeblock lang:javascript %}
function* g() {}
g.prototype.hello = function () {
    return 'hi';
}
let obj = g();
obj instanceof g;   // true
obj.hello();        // 'hi'
{% endcodeblock %}
让Generator函数返回一个正常对象的实例，既可以使用next方法，又可以获得正常的this：
* 方法一：
{% codeblock lang:javascript %}
function* F() {
    this.a = 1;
    yield this.b = 2;
    yield this.c = 3;
}
var obj = {};
var f = F.call(obj);    // obj 绑定this
f.next();   // { value : 2, done : false }
f.next();   // { value : 3, done : false }
f.next();   // { value : undefined, done : true }
obj.a;      // 1
obj.b;      // 2
obj.c;      // 3
{% endcodeblock %}
* 方法二：
{% codeblock lang:javascript %}
function* gen() {
    this.a = 1;
    yield this.b = 2;
    yield this.c = 3;
}
function F() {
    return gen.call(gen.prototype);
}
var f = new F();
f.next();   // { value : 2, done : false }
f.next();   // { value : 3, done : false }
f.next();   // { value : undefined, done : true }
f.a;        // 1
f.b;        // 2
f.c;        // 3
{% endcodeblock %}

### Generator函数与状态机
{% codeblock lang:javascript %}
var clock = function* () {
    while (true) {
        console.log('Tick');
        yield;
        console.log('Tock');
        yield;
    }
};
let c = clock();
c.next();   // Tick
c.next();   // Tock
c.next();   // Tick
{% endcodeblock %}
### 异步操作的同步化表达
{% codeblock lang:javascript %}
function* loadUI() {
    showLoadingScreen();
    yield loadUIDataAsynchronously();
    hideLoadingScreen();
}
var loader = loadUI();

//加载UI
loader.next();

//卸载UI
loader.next();
{% endcodeblock %}
用同步方式表达Generator部署AJAX操作：
{% codeblock lang:javascript %}
function* main() {
    var result = yield request('http://some.url');
    var resp = JSON.parse(result);
    console.log(resp.value);
}
function request(url) {
    makeAjaxCall(url, function(response) {
        it.next(response);
    });
}
var it = main();
it.next();
{% endcodeblock %}