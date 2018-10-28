---
title: Promise
date: 2018-08-31 16:00:39
tags:
- '学习'
- '整理'
categories:
- 'ECMAScript'
---
### Promise的特点
1. 对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：Pending（进行中）、Fulfilled（已成功）、Rejected（已失败）。只有异步操作的结果可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
2. 一旦状态改变就不会再变，任何时候得到的都是这个结果。Promise对象的状态改变只有两个可能：从Pending变为Fulfilled，从Pending变为Rejected。只要这两种情况发生，这时就成为resolve。就算改变已经发生，再对Promise对象添加回调函数，也会立即得到这个结果。与Event完全不同，Event一旦错过再监听是得不到结果的。

### 基本用法
{% codeblock lang:javascript %}
var promise = new Promise(function (resolve, reject) {
    //some code

    if (/*异步操作成功*/) {
        resolve(value);
    } else {
        reject(error);
    }
})
{% endcodeblock %}
resolve函数的作用是，将Promise对象的状态从Pending变为Resolved，在异步操作成功时调用，并将异步操作的结果作为参数传递出去。reject函数的作用是，将Promise对象的状态从Pending变为Rejected，将报出的错误传递出去。
Promise实例生成后，可以用then方法分别制定Resolve状态和Rejected状态的回调函数：
{% codeblock lang:javascript %}
promise.then(function (value) {
    //success
}, function (error) {
    //failure
})
{% endcodeblock %}
then方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为Resolved时调用，第二个回调是Promise对象的状态变为Rejected时调用。其中，第二个参数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数。
{% codeblock lang:javascript %}
function timeout(ms) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, ms, 'done');
    });
}
timeout(100).then(value => {
    console.log(value);
});
//'done'
{% endcodeblock %}
过了ms后，Promise实例的状态变为Resolved，触发then方法绑定的回调函数。
Promise新建后就会立即执行：
{% codeblock lang:javascript %}
let promise = new Promise(function (resolve, reject) {
    console.log('Promise');
    resolve();
});
promise.then(function () {
    console.log('Resolve');
});
console.log('hi');

//Promise
//hi
//Resolve
{% endcodeblock %}
then方法指定的回调函数将在当前脚本所有同步任务执行完成后才会执行，所以Resolve最后输出。
异步加载图片：
{% codeblock lang:javascript %}
function loadImageAsync(url) {
    return new Promise(function (resolve, reject) {
        var image = new Image();
        image.onload = function () {
            resolve(image);
        };
        image.onerror = function () {
            reject(new Error('Could not load image at ' + url));
        };
        image.src = url;
    });
}
{% endcodeblock %}
使用Promise实现AJAX：
{% codeblock lang:javascript %}
var getJSON = function (url) {
    var promise = new Promise(function (resolve, reject) {
        var client = new XMLHttpRequest();
        client.open('GET', url);
        client.onreadystatechange = handler;
        client.responseType = 'json';
        client.setRequestHeader('Accept', 'application/json');
        client.send();

        function handler() {
            if (this.readyState !== 4) {
                return;
            }
            if (this.status === 200) {
                resolve(this.response);
            } else {
                reject(new Error(this.statusText));
            }
        }
    });
    return promise;
};
getJSON('/posts.json').then(function (json) {
    console.log('Contents: ' + json);
}, function (error) {
    console.error('出错了', error);
});
{% endcodeblock %}
### 以Promise对象作为resolve的参数
{% codeblock lang:javascript %}
var p1 = new Promise(function (resolve, reject) {
    //...
});
var p2 = new Promise(function (resolve, reject) {
    //...
    resolve(p1);
});
{% endcodeblock %}
此时p1的状态决定了p2的状态。如果p1的状态是Pending，那么p2的回调就会等待p1的改变；如果p1的状态已经是Resolved或者Rejected，那么p2的回调函数就会立即执行。
{% codeblock lang:javascript %}
var p1 = new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('fail')), 3000);
});
var p2 = new Promise(function (resolve, reject) {
    setTimeout(() => resolve(p1), 1000);
});
p2.then(result => console.log(result)).catch(error => console.log(error));
{% endcodeblock %}
p1三秒后变为Rejected，p2在一秒后变为Resolved，由于p2返回的是另一个Promise，所以p2的状态无效，由p1的状态决定p2的状态。后面的then语句都变成针对p2的，再过两秒，p1变为Rejected，触发catch指定的回调函数。
**调用resolve或reject并不会结束Promise函数的执行**
因为立即resolve的Promise是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。所以最好在前面加上return语句。
### Promise.prototype.catch()
是.then(null, rejection)的别名，用于指定发生错误时的回调函数。
{% codeblock lang:javascript %}
getJSON('/posts.json').then(function (posts) {
    //...
}).catch(function (error) {
    //处理getJSON和前一个回调函数运行时发生的错误
})

p.then((val) => console.log('fulfilled:', val)).catch((err) => console.log('rejected:', err));
//等同于
p.then((val) => console.log('fulfilled:', val)).then(null, (err) => console.log('rejected', err));

var promise = new Promise(function (resolve, reject) {
    throw new Error('test');
});
promise.catch(function (error) {
    console.log(error);
});

//写法一
var promise = new Promise(function (resolve, reject) {
    try {
        throw new Error('test');
    } catch (e) {
        reject(e);
    }
});
promise.catch(function (error) {
    console.log(error);
});

//写法二
var promise = new Promise(function (resolve, reject) {
    reject(new Error('test'));
});
promise.catch(function (error) {
    console.log(error);
});
{% endcodeblock %}
比较可知reject方法的作用等同于抛出错误。
**如果Promise状态已经变成Resolved，再抛出错误是无效的**
{% codeblock lang:javascript %}
var promise = new Promise(function (resolve, reject) {
    resolve('ok');
    throw new Error('test');
});
promise.then(function (value) { console.log(value); }).catch(function (error) { console.log(error); }); //'ok'
{% endcodeblock %}
Promise对象的错误具有冒泡性质，会一直向后传递，直到被捕获为止。一般来说，应用Promise的catch方法。
与传统的try/catch不同的是，如果没有使用catch方法指定错误处理的回调函数，Promise对象抛出的错误不会传递到外层代码，不会有任何反应。需要注意的是，catch返回的还是一个Promise对象。
{% codeblock lang:javascript %}
var someAsyncThing = function () {
    return new Promise(function (resolve, reject) {
        resolve(x + 2); //ReferenceError
    });
};
someAsyncThing().catch(function (error) {
    console.log('error:', error);
}).then(function () {
    console.log('carry on');
});
//Error: ReferenceErro x is not defined
//carry on
{% endcodeblock %}
如果没有报错，会跳过catch方法。catch方法中还能抛出错误：
{% codeblock lang:javascript %}
var someAsyncThing = function () {
    return new Promise(function (resolve, reject) {
        resolve(x + 2); //ReferenceError
    });
};
someAsyncThing().then(function () {
    return someOtherAsyncThing();
}).catch(function (error) {
    console.log(error);
    y + 2;  //ReferenceError
}).catch(function (error) {
    console.log('carry on', error);
});
//x in not defined
//carry on y is not defined
{% endcodeblock %}
### Promise.all
将多个Promise对象包装成一个新的实例。
{% codeblock lang:javascript %}
var p = Promise.all(p1, p2, p3);
{% endcodeblock %}
p的状态由p1、p2、p3决定：
1. 只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。
2. 只要p1、p2、p3有一个被Rejected，p的状态就变为Rejected，此时第一个被Rejected的实例的返回值会传递给p的回调函数。

{% codeblock lang:javascript %}
const databasePromise = connectDatabse();
const booksPromise = databasePromise.then(findAllBooks);
const userPromise = databasePromise.then(getCurrentUser);
Promise.all([booksPromise, userPromise]).then(([books, user]) => pickTopRecommentations(books, user));
{% endcodeblock %}
只有booksPromise和userPromise结果都返回，才会触发pickTopRecommentations回调函数。
**如果作为参数的Promise实例自身定义了catch方法，那么它被rejected时并不会触发Promise.all的catch方法**
{% codeblock lang:javascript %}
const p1 = new Promise((resolve, reject) => {
    resolve('hello');
}).then(result => result).catch(e => e);
const p2 = new Promise((resolve, reject) => {
    throw new Error('test');
}).then(result => result).catch(e => e);
Promise.all([p1, p2]).then(result => console.log(result)).catch(e => console.log(e));
//['hello', Error: test]
{% endcodeblock %}
如果p2没有自己的catch方法，就会调用Promise.all的catch方法：
{% codeblock lang:javascript %}
const p1 = new Promise((resolve, reject) => {
    resolve('hello');
}).then(result => result);
const p2 = new Promise((resolve, reject) => {
    throw new Error('test');
}).then(result => result);
Promise.all([p1, p2]).then(result => console.log(result)).catch(e => console.log(e));
//Error: test
{% endcodeblock %}
### Promise.race
将多个Promise实例包装成一个新的实例。
{% codeblock lang:javascript %}
var p = Promise.race([p1, p2, p3]);
{% endcodeblock %}
只要p1、p2、p3有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的实例的返回值就传给p的回调函数。
{% codeblock lang:javascript %}
const p = Promise.race([
    fetch('/resource-that-may-take-a-while'),
    new Promise(function (resolve, reject) {
        setTimeout(() => reject(new Error('request timeout')), 5000);
    })
]);
p.then(response => console.log(response));
p.catch(error => console.log(error));
//五秒内fetch无法返回变量，p的状态就变为Rejected，从而触发catch方法的回调函数
{% endcodeblock %}
### Promise.resolve
* 参数是一个Promise实例，不做任何修改，返回这个实例
* 参数是一个thenable对象，将这个对象转为Promise对象，然后立即执行thenable对象的then方法
{% codeblock lang:javascript %}
let thenable = {
    then : function (resolve, reject) {
        resolve(42);
    }
};
let p1 = Promise.resolve(thenable);
p1.then(function (value) {
    console.log(value);
});
//42
{% endcodeblock %}
* 参数根本不是具有then方法的对象或者不是对象，返回一个新的Promise对象，状态为Resolved
{% codeblock lang:javascript %}
var p = Promise.resolve('hello');
p.then(function (s) {
    console.log(s);
});
//'hello'
{% endcodeblock %}
* 不带有任何参数，直接返回一个Resolved状态的对象
* Promise.reject 返回一个状态为Rejected的Promise对象

**立即resolve的Promise对象是在本轮事件循环结束时，而不是在下次事件循环开始时**
### done
只要最后一个方法抛出错误，都有可能无法捕捉到，为此可以配置一个done方法。
{% codeblock lang:javascript %}
asyncFunc().then(f1).catch(r1).then(f2).done();
{% endcodeblock %}
实现的代码如下：
{% codeblock lang:javascript %}
Promise.prototype.done = function (onFulfilled, onRejected) {
    this.then(onFulfilled, onRejected).catch(function (reason) {
        setTimeout(() => { throw reason; }, 0); //抛出一个全局错误
    })
}
{% endcodeblock %}
### finally
与done最大的区别在于，接受一个普通的回调函数作为参数，该函数不管怎样都执行。
{% codeblock lang:javascript %}
server.listen(0).then(function () {
    // run test
}).finally(server.stop);
{% endcodeblock %}
实现代码如下：
{% codeblock lang:javascript %}
Promise.prototype.finally = function (callback) {
    let P = this.constructor;
    return this.then(
        value => P.resolve(callback()).then(() => value),
        reason => P.resolve(callback()).then(() => { throw reason; })
    );
};
{% endcodeblock %}
eg: 加载图片
{% codeblock lang:javascript %}
const preloadImage = function (path) {
    return new Promise(function (resolve, reject) {
        var image = new Image();
        image.onload = resolve;
        image.onerror = reject;
        image.src = path;
    });
};
{% endcodeblock %}
### Generator函数与Promise结合
使用Generator函数管理流程，遇到异步操作时通常返回一个Promise对象。
{% codeblock lang:javascript %}
function getFoo() {
    return new Promise(function (resolve, reject) {
        resolve('foo');
    });
}
var g = function* () {
    try {
        var foo = yield getFoo();
        console.log(foo);
    } catch (e) {
        console.log(e);
    }
};
function run(generator) {
    var it = generator();
    function go(result) {
        if (result.done) return result.value;
        return result.value.then(function (value) {
            return go(it.next(value));
        }, function (error) {
            return go(it.throw(error));
        });
    }
    go(it.next());
}
run(g); // 用run处理Promise对象，并调用下一个next方法
{% endcodeblock %}