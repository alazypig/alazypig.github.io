---
title: Reflect
date: 2018-08-31 14:56:58
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
1. 从Reflect对象上可以获得语言内部的方法
2. 修改某些Object方法的返回结果，让其变得更合理。比如Object.defineProperty在无法定义属性时会抛出一个错误，而Reflect.defineProperty则会返回false
3. 让Object操作都变成函数行为。
4. 只要是Proxy对象的方法，就能在Reflect对象上找到相应的方法，无论Proxy怎么修改默认行为，总可以在Reflect上获取默认行为

### 静态方法
* Reflect.apply(target, thisArg, args)
等同于Function.prototype.apply.call(func, thisArg, args)，用于绑定this对象后执行给定函数。
* Reflect.construct(target, args)
等同于new target(...args)，提供了一种不使用new来调用构造函数的方法：
{% codeblock lang:javascript %}
function Greeting(name) {
    this.name = name;
}

//new的写法
const instance = new Greeting('张三');

//Reflect.construct写法
const instance = Reflect.construct(Greeting, ['张三']);
{% endcodeblock %}
* Reflect.get(target, name, receiver)
查找并返回target的name属性，如果没有返回undefined。
{% codeblock lang:javascript %}
let obj = {
    foo : 1,
    bar : 2,
    get baz() {
        return this.foo + this.bar;
    }
}
Reflect.get(obj, 'foo');    //1
Reflect.get(obj, 'baz');    //3
{% endcodeblock %}
如果name属性部署了getter，则getter的this绑定receiver：
{% codeblock lang:javascript %}
let obj = {
    foo : 1,
    bar : 2,
    get gaz() {
        return this.foo + this.bar;
    }
};
let myobj = {
    foo : 2,
    bar : 4,
}
Reflect.get(obj, 'gaz', myobj); //myobj.foo + myobj.bar 6
{% endcodeblock %}
**如果第一个参数不是object，会报错**
Reflect.set会触发Proxy.defineProperty拦截：
{% codeblock lang:javascript %}
let p = {
    a : 'a'
};
let handler = {
    set(target, key, value, receiver) {
        console.log('set');
        Reflect.set(target, key, value, receiver);
    },
    defineProperty(target, key, attribute) {
        console.log('defineProperty');
        Reflect.defineProperty(target, key, attribute);
    }
};
let obj = new Proxy(p, handler);
obj.a = 'A';
//set
//defineProperty
{% endcodeblock %}
* Reflect.set(target, name, value, receiver)
设置target的name属性等于value。
{% codeblock lang:javascript %}
let obj = {
    foo : 1,
    set bar(value) {
        return this.foo = value;
    }
};
obj.foo;    //1
Reflect.set(obj, 'foo', 2);
obj.foo;    //2
{% endcodeblock %}
如果name属性设置了setter，则setter的this绑定receiver：
{% codeblock lang:javascript %}
let obj = {
    foo : 1,
    set bar(value) {
        return this.foo = value;
    }
};
let myobj = {
    foo : 0
};
Reflect.set(obj, 'bar', 4, myobj);
myobj.foo;  //4
{% endcodeblock %}
**如果第一个参数不是object，会报错**
* Reflect.defineProperty(target, name, descriptor)
用来定义对象的属性。
* Reflect.deleteProperty(target, name)
等同于delete obj[name]，用于删除对象的属性，返回一个布尔值，删除成功返回true，否则返回false。
* Reflect.has(target, name)
对应name in target中的in运算符，如果第一个参数不是对象，Reflect.has和in都会报错。
* Reflect.ownKeys(target)
返回对象的所有属性，包括Symbol属性。
* Reflect.isExtensible(target)
返回一个布尔值，表示当前对象是否可拓展。
* Reflect.preventExtensions(target)
用于使一个对象变为不可拓展的，返回一个布尔值，代表是否成功。
* Reflect.getOwnPropertyDescriptor(target, name)
基本等同于Object.getOwnPropertyDescriptor(target, propertyKey)，用于获得指定属性的描述对象。
* Reflect.getPrototypeOf(target)
用于读取对象的__prop__属性，对应Object.getPrototypeOf(obj)。
* Reflect.setPrototypeOf(target, prototype)
用于设置对象的__prop__属性，返回第一个参数对象。

### 用Proxy实现观察者模式
Observe mode指的是函数自动观察数据对象的模式，一旦对象有变化，函数就会自动执行。
思路：使用observable和observe这两个函数，observable函数返回一个原始对象的Proxy代理，拦截赋值操作，触发充当观察者的各个函数。
{% codeblock lang:javascript %}
const queueObservers = new Set();
const observe = fn => queueObservers.add(fn);
const observable = obj => new Proxy(obj, { set });  //拦截set

function set(target, key, value, receiver) {
    const result = Reflect.set(target, key, value, receiver);   //完成原始操作
    queueObservers.forEach(observer => observer());
    return result;
}

const person = observable({
    name : '张三',
    age : 20
});
function print() {
    console.log(`${person.name}， ${person.age}`);
}

observe(print);
person.name = '李四';   //李四，20
{% endcodeblock %}