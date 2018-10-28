---
title: Proxy
date: 2018-08-20 19:10:16
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
Proxy用于修改某些操作的默认行为，等同与在语言层面做出修改，属于一种meta programming。
{% codeblock lang:javascript %}
let obj = new Proxy({}, {
    get: function (target, key, receiver) {
        console.log(`getting ${key}`);
        return Reflect.get(target, key, receiver);
    },
    set: function (target, key, receiver) {
        console.log(`setting ${key}`);
        return Reflect.set(target, key, receiver);
    }
});
obj.count = 1;
//setting count
obj.count;
//getting count
{% endcodeblock %}
ES6提供Proxy构造函数，用于生成Proxy实例。
{% codeblock lang:javascript %}
let proxy = new Proxy(target, handler);
{% endcodeblock %}
将Proxy对象设置到object.proxy属性，从而可以在object对象上调用：
{% codeblock lang:javascript %}
let obj = { proxy: new Proxy(target, handler) };
{% endcodeblock %}
Proxy实例也可以作为其他对象的原型对象：
{% codeblock lang:javascript %}
let proxy = new Proxy({}, {
    get: function (target, handler) {
        return 2;
    }
});
let obj = Object.create(proxy);
obj.time;   //2
{% endcodeblock %}
同一个Proxy可以设置多个拦截属性：
{% codeblock lang:javascript %}
let handler = {
    get: function (target, name) {
        if (name === 'prototype') {
            return Object.prototype;
        }
        return 'Hello, ' + name;
    },
    apply: function (target, thisBinding, args) {
        return args[0];
    },
    construct: function (target, args) {
        return { value: args[1] };
    }
};
let fproxy = new Proxy(function (x, y) {
    return x + y;
}, handler);

fproxy(1, 2);                           //1
new fproxy(1, 2);                       //2
fproxy.prototype === Object.prototype;  //true
fproxy.foo;                             //'hello, foo'
{% endcodeblock %}
### Proxy方法
* get(target, propKey, receiver)
拦截对象的属性读取，如proxy.foo和proxy['foo']。最后一个参数是可选的。
{% codeblock lang:javascript %}
let person = {
    name : '张三'
};
let proxy = new Proxy(person, {
    get : function (target, property) {
        if (property in target) {
            return target[property];
        } else {
            throw new ReferenceError('Property \"' + property + '\" does not exist.');
        }
    }
});
proxy.name; //张三
proxy.age;  //ReferenceError
{% endcodeblock %}
get方法可以继承：
{% codeblock lang:javascript %}
let proto = new Proxy({}, {
    get(target, propertyKey, receiver) {
        console.log('get ' + propertyKey);
        return target[propertyKey];
    }
});
let obj = Object.create(proto);
obj.aaa;    //get aaa
{% endcodeblock %}
使用get实现负数索引：
{% codeblock lang:javascript %}
function createArray(...elements) {
    let handler = {
        get(target, propKey, receiver) {
            let index = Number(propKey);
            if (index < 0) {
                propKey = String(target.length + index);
            }
            return Reflect.get(target, propKey, receiver);
        }
    };
    let target = [];
    target.push(...elements);
    return new Proxy(target, handler);
}
let arr = createArray('a', 'b', 'c');
arr[-1];    //'c'
{% endcodeblock %}
将get转为执行某个函数，实现属性的链式操作：
{% codeblock lang:javascript %}
var pipe = (function () {
    return function (value) {
        var funcStack = [];
        var oproxy = new Proxy({}, {
            get : function (pipeObject, fnName) {
                if (fnName === 'get') {
                    return funcStack.reduce(function (val, fn) {    //reduce接受一个函数作为累加器，从左到右缩减，最终计算为一个值
                        return fn(val);
                    }, value);
                }
                funcStack.push(window[fnName]);
                return oproxy;
            }
        });
        return oproxy;
    }
}());
var double = n => n * 2;
var pow = n => n * n;
var reverseInt = n => n.toString().split('').reverse().join('') | 0;
pipe(3).double.pow.reverseInt.get;  //63
{% endcodeblock %}
* set(target, propKey, value, receiver)
拦截对象的属性设置，如proxy.foo = 1，返回一个布尔值。
{% codeblock lang:javascript %}
let validator = {
    set : function (obj, prop, value) {
        if (prop === 'age') {
            if (!Number.isInteger(value)) {
                throw new TypeError('The age is not an integer');
            }
            if (value > 200) {
                throw new RangeError('The age seems invalid');
            }
        }
        //age < 200直接保存
        obj[prop] = value;
    }
};
let person = new Proxy({}, validator);
person.age = 100;
person.age = 201;   //RangeError
person.age = 'tom'; //TypeError
{% endcodeblock %}
给对象设置内部属性：
{% codeblock lang:javascript %}
var handler = {
    get(target, key) {
        invariant(key, 'get');
        return target[key];
    },
    set(target, key, value) {
        invariant(key, 'set');
        target[key] = value;
        return true;
    }
};
function invariant(key, action) {
    if (key[0] === '_') {
        throw new Error(`Invalid attempt to ${action} private "${key}" property`);
    }
}
var target = {};
var proxy = new Proxy(target, handler);
proxy._temp;        //Error
proxy._temp = 2;    //Error
{% endcodeblock %}
* has(target, propKey)
拦截propKey in proxy的操作，返回一个布尔值。has拦截对for...in循环不生效
* deleteProperty(target, propKey)
拦截delete proxy[propKey]的操作，返回一个布尔值。如果这个方法抛出错误或者返回false，当前属性就不能被delete删除。
{% codeblock lang:javascript %}
var handler = {
    deleteProperty(target, key) {
        invariant(key, 'delete');
        return true;
    }
};
function invariant(key, action) {
    if (key[0] === '_') {
        throw new Error(`Invalid attempt to ${action} private "${key}" property`);
    }
}
var target = { _temp : 'test' };
var proxy = new Proxy(target, handler);
delete proxy._temp; //Error
{% endcodeblock %}
* ownKeys(target)
拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)，返回一个数组。该方法返回所有属性名，而
Object.keys()返回结果仅包括目标对象和自身的可遍历属性。
* getOwnPropertyDescriptor(target, propKey)
拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象或者undefined。
{% codeblock lang:javascript %}
var handler = {
    getOwnPropertyDescriptor(target, key) {
        if (key[0] === '_') {
            return;
        }
        return Object.getOwnPropertyDescriptor(target, key);
    }
};
var target = { _foo : 'foo', bar : 'bar' };
var proxy = new Proxy(target, handler);
Object.getOwnPropertyDescriptor(proxy, '_foo'); //undefined
Object.getOwnPropertyDescriptor(proxy, 'bar');  //{ value : 'bar', writable : true, enumerable : true, configurable : true }
{% endcodeblock %}
* defineProperty(target, propKey, propDesc)
拦截Objec.defineProperty:
{% codeblock lang:javascript %}
var handler = {
    defineProperty(target, key, descriptor) {
        return false;
    }
};
var target = {};
var proxy = new Proxy(target, handler);
proxy.foo = 'bar';
proxy.foo;  //undefined
{% endcodeblock %}
* preventExtensions(target)
拦截Object.preventExtensions，必须返回布尔值，否则会强制转换为布尔值。
* getPrototypeOf(target)
拦截获取对象原型
* isExtensible(target)
拦截Objec.isExtensible操作
* setPrototypeOf(target, proto)
拦截Object.setPrototypeOf方法
* apply(target, object, args)
拦截函数的调用、call和apply操作，
{% codeblock lang:javascript %}
var target = function () { return 'test'; }
var handler = {
    apply : function () {
        return 'apply';
    }
};
let p = new Proxy(target, handler);
p();    //apply
{% endcodeblock %}
* construct(target, args)
拦截new命令，返回的必须是对象，否则会报错：
{% codeblock lang:javascript %}
var p = new Proxy(function () {}, {
    construct : function (target, args) {
        console.log('called ' + args.join(','));
        return { value : args[0] * 10 };
    }
});
(new p(1)).value;   
//called 1
//10
{% endcodeblock %}
### Proxy.revocable()
返回一个可取消的Proxy实例：
{% codeblock lang:javascript %}
let target = {};
let handler = {};
let { proxy, revoke } = Proxy.revocable(target, handler);
proxy.foo = 123;
proxy.foo;  //123

revoke();
proxy.foo;  //TypeError
{% endcodeblock %}
执行revoke函数后再访问Proxy实例，就会抛出一个错误。
### this问题
在Proxy代理下，目标对象内部的this关键字会指向Proxy代理：
{% codeblock lang:javascript %}
const _name = new WeakMap();
class Person {
    constructor(name) {
        _name.set(this, name);
    }
    get(name) {
        return _name.get(this);
    }
}
const jane = new Person('Jane');
jane.name;  //'Jane'
const proxy = new Proxy(jane, {});
proxy.name; //undefined
{% endcodeblock %}
此外，有些原生对象内部属性只有通过正确的this才能获取，所以Proxy也无法代理这些原生对象的属性
{% codeblock lang:javascript %}
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);
proxy.getDate();//TypeError: This is not a Date Object
{% endcodeblock %}
这时，this绑定原始对象就可以解决
{% codeblock lang:javascript %}
const target = new Date('2018-8-31');
const handler = {
    get(target, prop) {
        if (prop === 'getDate') {
            return target.getDate.bind(target);
        }
        return Reflect.get(target, prop);
    }
};
const proxy = new Proxy(target, handler);
proxy.getDate();    //31
{% endcodeblock %}