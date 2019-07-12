## promise简介

Promise的出现，原本是为了解决回调地狱的问题。所有人在讲解Promise时，都会以一个ajax请求为例，此处我们也用一个简单的ajax的例子来带大家看一下Promise是如何使用的。

ajax请求的传统写法：

```javascript
getData(method, url, successFun, failFun){
  var xmlHttp = new XMLHttpRequest();
  xmlHttp.open(method, url);
  xmlHttp.send();
  xmlHttp.onload = function () {
    if (this.status == 200 ) {
      successFun(this.response);
    } else {
      failFun(this.statusText);
    }
  };
  xmlHttp.onerror = function () {
    failFun(this.statusText);
  };
}
```

改为promise后的写法：

```javascript
getData(method, url){
  var promise = new Promise(function(resolve, reject){
    var xmlHttp = new XMLHttpRequest();
    xmlHttp.open(method, url);
    xmlHttp.send();
    xmlHttp.onload = function () {
      if (this.status == 200 ) {
        resolve(this.response);
      } else {
        reject(this.statusText);
      }
    };
    xmlHttp.onerror = function () {
      reject(this.statusText);
    };
  })
  return promise;
}

getData('get','www.xxx.com').then(successFun, failFun)
```

很显然，我们把异步中使用回调函数的场景改为了.then()、.catch()等函数链式调用的方式。基于promise我们可以把复杂的异步回调处理方式进行模块化。

下面，我们就来介绍一下Promise到底是个什么东西？它是如何做到的？

## Promise的状态

其实promise原理说起来并不难，它内部有**三个状态，分别是pending，fulfilled和rejected** 。

pending是对象创建后的初始状态，当对象fulfill（成功）时变为fulfilled，当对象reject（失败）时变为rejected。且只能从pengding变为fulfilled或rejected ，而不能逆向或从fulfilled变为rejected 、从rejected变为fulfilled。如图所示：



![img](https:////upload-images.jianshu.io/upload_images/12665637-9ee2ecf3ca4f035e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/690/format/webp)



## Promise实例方法介绍

Promise对象拥有**两个实例方法then()和catch()**。

从前面的例子中可以看到，成功和失败的回调函数我们是通过then()添加，在promise状态改变时分别调用。promise构造函数中通常都是异步的，所以then方法往往都先于resolve和reject方法执行。所以promise内部需要有一个存储fulfill时调用函数的数组和一个存储reject时调用函数的数组。

从上面的例子中我们还可以看到then方法可以接收两个参数，且通常都是函数（非函数时如何处理下一篇文章中会详细介绍）。第一个参数会添加到fulfill时调用的数组中，第二个参数添加到reject时调用的数组中。当promise状态fulfill时，会把resolve(value)中的value值传给调用的函数中，同理，当promise状态reject时，会把reject(reason)中的reason值传给调用的函数。例：

```javascript
var p = new Promise(function(resolve, reject){
    resolve(5)
}).then(function(value){
    console.log(value) //5
})

var p1 = new Promise(function(resolve, reject){
    reject(new Error('错误'))
}).then(function(value){
    console.log(value)
}, function(reason){
    console.log(reason) //Error: 错误(…)
})
```

then方法会返回一个新的promise，下面的例子中p == p1将返回false,说明p1是一个全新的对象。

```javascript
var p = new Promise(function(resolve, reject){
    resolve(5)
})
var p1 = p.then(function(value){
    console.log(value)
})
p == p1 // false
```

这也是为什么then是可以链式调用的，它是在新的对象上添加成功或失败的回调，这与jQuery中的链式调用不同。

那么新对象的状态是基于什么改变的呢？是不是说如果p的状态fulfill，后面的then创建的新对象都会成功；或者说如果p的状态reject，后面的then创建的新对象都会失败？

```javascript
var p = new Promise(function(resolve, reject){
    resolve(5)
})
var p1 = p.then(function(value){
    console.log(value)   // 5
}).then(function(value){
    console.log('fulfill ' + value)   // fulfill undefined
}, function(reason){
    console.log('reject ' + reason)   
})
```

上面的例子会打印出5和"fulfill undefined"说明它的状态变为成功。那如果我们在p1的then方法中抛出异常呢？

```javascript
var p = new Promise(function(resolve, reject){
    resolve(5)
})
var p1 = p.then(function(value){
    console.log(value)   // 5
    throw new Error('test')
}).then(function(value){
    console.log('fulfill ' + value)
}, function(reason){
    console.log('reject ' + reason)   // reject Error: test
})
```

理所当然，新对象肯定会失败。

反过来如果p失败了，会是什么样的呢？

```javascript
var p = new Promise(function(resolve, reject){
    reject(5)
})
var p1 = p.then(undefined, function(value){
    console.log(value)   // 5
}).then(function(value){
    console.log('fulfill ' + value)   // fulfill undefined
}, function(reason){
    console.log('reject ' + reason)
})
```

说明新对象状态不会受到前一个对象状态的影响。

再来看如下代码：

```javascript
var p = new Promise(function(resolve, reject){
    reject(5)
})
var p1 = p.then(function(value){
    console.log(value) 
})
var p2 = p1.then(function(value){
    console.log('fulfill ' + value)
}, function(reason){
    console.log('reject ' + reason)   // reject 5
})
```

我们发现p1的状态变为rejected，从而触发了then方法第二个参数的函数。这似乎与我们之前提到的有差异啊，p1的状态受到了p的状态的影响。

再来看一个例子：

```javascript
var p = new Promise(function(resolve, reject){
    resolve(5)
})
var p1 = p.then(undefined, function(value){
    console.log(value) 
})
var p2 = p1.then(function(value){
    console.log('fulfill ' + value)   // fulfill 5
}, function(reason){
    console.log('reject ' + reason)   
})
```

细心的人可能会发现，该例子中then第一个参数是undefined，且value值5被传到了p1成功时的回调函数中。上面那个例子中then的第二个参数是undefined，同样reason值也传到了p1失败时的回调函数中。这是因当对应的参数不为函数时，会将前一promise的状态和值传递下去。

promise含有一个实例方法catch，从名字上我们就看得出来，它和异常有千丝万缕的关系。其实catch(onReject)方法等价于then(undefined, onReject)，也就是说如下两种情况是等效的。

```javascript
new Promise(function(resolve, reject){
    reject(new Error('error'))
}).then(undefined, function(reason){
    console.log(reason) // Error: error(…)
})

new Promise(function(resolve, reject){
    reject(new Error('error'))
}).catch(function(reason){
    console.log(reason) // Error: error(…)
})
```

我们提到参数不为函数时会把值和状态传递下去。所以我们可以在多个then之后添加一个catch方法，这样前面只要reject或抛出异常，都会被最后的catch方法处理。

```javascript
new Promise(function(resolve, reject){
    resolve(5)
}).then(function(value){
    taskA()
}).then(function(value){
    taskB()
}).then(function(value){
    taskC()
}).catch(function(reason){
    console.log(reason)
})
```

## Promise的静态方法

Promise还有**四个静态方法**，分别是**resolve、reject、all、race**，下面我们一一介绍。

除了通过new Promise()的方式，我们还有两种创建Promise对象的方法：
 **Promise.resolve() 它相当于创建了一个立即resolve的对象。如下两段代码作用相同：**

```javascript
Promise.resolve(5)

new Promise(function(resolve){
    resolve(5)
})
```

它使得promise对象直接resolve，并把5传到后面then添加的成功函数中。

```javascript
Promise.resolve(5).then(function(value){
    console.log(value) // 5
})
```

**Promise.reject()** 很明显它相当于创建了一个立即reject的对象。如下两段代码作用相同：

```javascript
Promise.reject(new Error('error'))

new Promise(function(resolve, reject){
    reject(new Error('error'))
})
```

它使得promise对象直接reject，并把error传到后面catch添加的函数中

```javascript
Promise.reject(new Error('error')).catch(function(reason){
    console.log(reason) // Error: error(…)
})
```

**Promise.all()** 它接收一个promise对象组成的数组作为参数，并返回一个新的promise对象。

当数组中所有的对象都resolve时，新对象状态变为fulfilled，所有对象的resolve的value依次添加组成一个新的数组，并以新的数组作为新对象resolve的value，例：

```javascript
Promise.all([Promise.resolve(5), 
  Promise.resolve(6), 
  Promise.resolve(7)]).then(function(value){
    console.log('fulfill', value)  // fulfill [5, 6, 7]
}, function(reason){
    console.log('reject',reason)
})
```

当数组中有一个对象reject时，新对象状态变为rejected，并以当前对象reject的reason作为新对象reject的reason。

```javascript
Promise.all([Promise.resolve(5), 
  Promise.reject(new Error('error')), 
  Promise.resolve(7),
  Promise.reject(new Error('other error'))
  ]).then(function(value){
    console.log('fulfill', value)
}, function(reason){
    console.log('reject', reason)  // reject Error: error(…)
})
```

那当数组中，传入了非promise对象会如何呢？

```javascript
Promise.all([Promise.resolve(5), 
  6,
  true,
  'test',
  undefined,
  null,
  {a:1},
  function(){},
  Promise.resolve(7)
  ]).then(function(value){
    console.log('fulfill', value)  // fulfill [5, 6, true, "test", undefined, null, Object, function, 7]
}, function(reason){
    console.log('reject', reason)
})
```

我们发现，当传入的值为数字、boolean、字符串、undefined、null、{a:1}、function(){}等非promise对象时，会依次把它们添加到新对象resolve时传递的数组中。

那数组中的多个对象是同时调用，还是一个接一个的依次调用呢？我们再看个例子

```javascript
function timeout(time) {
    return new Promise(function (resolve) {
        setTimeout(function () {
            resolve(time);
        }, time);
    });
}
console.time('promise')
Promise.all([
    timeout(10),
    timeout(60),
    timeout(100)
]).then(function (values) {
    console.log(values); [10, 60, 100]
    console.timeEnd('promise');   // 107ms 
});
```

由此我们可以看出，传入的多个对象几乎是同时执行的，因为总的时间略大于用时最长的一个对象resolve的时间。

**Promise.race()** 它同样接收一个promise对象组成的数组作为参数，并返回一个新的promise对象。

与Promise.all()不同，它是在数组中有一个对象（最早改变状态）resolve或reject时，就改变自身的状态，并执行响应的回调。

```javascript
Promise.race([Promise.resolve(5), 
  Promise.reject(new Error('error')), 
  Promise.resolve(7)]).then(function(value){
    console.log('fulfill', value)  // fulfill 5
}, function(reason){
    console.log('reject',reason)
})

Promise.race([Promise.reject(new Error('error')), 
  Promise.resolve(7)]).then(function(value){
    console.log('fulfill', value) 
}, function(reason){
    console.log('reject',reason) //reject Error: error(…)
})
```

且当数组中有非异步Promise对象或有数字、boolean、字符串、undefined、null、{a:1}、function(){}等非Promise对象时，都会直接以该值resolve。

```javascript
Promise.race([new Promise((resolve)=>{
    setTimeout(()=>{
        resolve(1)
    },100)}),
  Promise.resolve(5), 
  "test",
  Promise.reject(new Error('error')), 
  Promise.resolve(7)]).then(function(value){
    console.log('fulfill', value)  // fulfill 5
}, function(reason){
    console.log('reject',reason)
})
// fulfill 5
```

**数组中第一个元素是异步的Promise，第二个是非异步Promise，会立即改变状态，所以新对象会立即改变状态并把5传递给成功时的回调函数。**

那么问题又来了，既然数组中第一个元素成功或失败就会改变新对象的状态，那数组中后面的对象是否会执行呢?

```javascript
function timeout(time) {
    return new Promise(function (resolve) {
        setTimeout(function () {
            console.log(time)
            resolve(time);
        }, time);
    });
}
console.time('promise')
Promise.race([
    timeout(10),
    timeout(60),
    timeout(100)
]).then(function (values) {
    console.log(values); [10, 60, 100]
    console.timeEnd('promise');   // 107ms
});

// 结果依次为
// 10
// 10
// promise: 11.1ms
// 60
// 100
```

说明即使新对象的状态改变，数组中后面的promise对象还会执行完毕，其实Promise.all()中即使前面reject了，所有的对象也都会执行完毕。规范中，promise对象执行是不可以中断的。

## 补充

promise对象即使立马改变状态，它也是异步执行的。如下所示：

```javascript
Promise.resolve(5).then(function(value){
  console.log('后打出来', value)
});
console.log('先打出来')

// 结果依次为
// 先打出来
// 后打出来 5
```

但还有一个有意思的例子，如下：

```javascript
setTimeout(function(){console.log(4)},0);
new Promise(function(resolve){
    console.log(1)
    for( var i=0 ; i<10000 ; i++ ){
        i==9999 && resolve()
    }
    console.log(2)
}).then(function(){
    console.log(5)
});
console.log(3);
```

结果是 1 2 3 5 4，命名4是先添加到异步队列中的，为什么结果不是1 2 3 4 5呢？这个涉及到Event loop，[我之前文章](https://www.jianshu.com/p/16a347d3add1)讲过。