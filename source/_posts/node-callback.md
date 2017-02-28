---
title: Nodejs解决回调地狱
date: 2017-02-14 10:21:02
tags: 
- nodejs 
- js
- async
- Generator
---
Nodejs是单线程运行的，很多时候需要写回调方法，但当一些业务逻辑或io操作需要顺序执行的时候，会产生层层回调。

```javascript
asyncFun1(function(err, a) {
    // do something with a in function 1
    asyncFun2(function(err, b) {
        // do something with b in function 2
        asyncFun3(function(err, c) {
            // do something with c in function 3
        });
    });
});
```

这样的代码难以阅读和维护。

#### 解决方案

有很多处理回调地狱的解决方案。

##### 具名函数

可以使用具名函数将代码段抽离出来，但这样需要些很多**function**，并且可读性还是很差，并没有多大改进，写起来也麻烦。

```javascript
function fun3(err, c) {
    // do something with c in function 3
}
function fun2(err, b) {
    // do something with b in function 2 
    asyncFun3(fun3);
}
function fun1(err, a) {
    // do something with a in function 1
    asyncFun2(fun2);
}
asyncFun1(fun1);
```

##### Promise

进阶一级的使用Promise或者链式Promise，但是还是需要不少的回调，虽然没有了嵌套。

```javascript
asyncFun1().then(function(a) {
    // do something with a in function 1
    asyncFun2();
}).then(function(b) {
    // do something with b in function 2
    asyncFun3();
}).then(function(c) {
    // do somethin with c in function 3
});
```

##### Async库

虽然Async库比较强大，但需要引入额外的库，代码也不是很直观。

```javascript
async.series([
    function(callback) {
        // do some stuff ...
        callback(null, 'one');
    },
    function(callback) {
        // do some more stuff ...
        callback(null, 'two');
    }
],
// optional callback
function(err, results) {
    // results is now equal to ['one', 'two']
});
```

##### Generator函数

[Generator](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function*)函数的作用其实就是将改方法里的代码分块，可以控制每一块代码块什么时候执行，也就是将函数的控制权交给别人，层层回调出现的原因其实就是需要顺序执行，但是这种顺序执行是出现在线程任意的执行片段的，**并且不能阻塞线程**，Generator函数可以实现在线程任意的执行片段，通知该Generator函数从之前停下的地方继续执行，达到异步代码，看起来完全像是同步代码的效果。

```javascript
function asyncFn (callback) {
  setTimeout(function() {
  	callback('haha');
  }, 1000);
}

var gen = function* (){
  var r1 = yield asyncFn
  var r2 = yield asyncFn
}

//执行
var g = gen();

///执行器
var r1 = g.next();
r1.value(function(data){
  var r2 = g.next(data)
  r2.value(function(data){
    g.next(data)
  })
})
```

上面的代码是手动控制Generator函数的例子，可以看到*gen*方法中两个*asyncFn*是顺序的写法，可读性高，但是执行器还是有回调，不过可以通过递归函数来解决。

```javascript
//执行器
function run(fn) {
  var gen = fn();
  
  function next(err,data){
    var result = gen.next(data)
    if(result.done) return;
    result.value(next)
  }
  
  next()
}

var gen = function* (){
  var r1 = yield asyncFn
  var r2 = yield asyncFn
}

//执行
run(gen);
```

控制器是可以复用的，所以有了以后只需要写Generator函数，然后调用一下 `run(gen)` 即可。

##### co模块

[co](https://github.com/tj/co)模块就是一个Generator函数的执行器，使用co模块的前提是，Generator函数的*yield*命令后面只能使用*[Thunk](http://www.ruanyifeng.com/blog/2015/05/thunk.html)*函数或Promise对象。

##### async函数

ES7提供了[async](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)函数，一句话，async函数就是Generator函数的语法糖。上面的例子改写成async函数就是下面这样

```javascript
var test = async function (){
  var r1 = await asyncFn
  var r2 = await asyncFn
}

//执行
test();
```

没有任何多余的`run(gen)`语句。

##### async函数的实现

async 函数的实现，就是将 Generator 函数和自动执行器，包装在一个函数里。

```javascript
async function fn(args){
  // ...
}

// 等同于

function fn(args){ 
  return spawn(function*() {
    // ...
  }); 
}
```

所有的 async 函数都可以写成上面的第二种形式，其中的 spawn 函数就是自动执行器，实现如下：

```javascript
function spawn(genF) {
  return new Promise(function(resolve, reject) {
    var gen = genF();
    function step(nextF) {
      try {
        var next = nextF();
      } catch(e) {
        return reject(e); 
      }
      if(next.done) {
        return resolve(next.value);
      } 
      Promise.resolve(next.value).then(function(v) {
        step(function() { return gen.next(v); });      
      }, function(e) {
        step(function() { return gen.throw(e); });
      });
    }
    step(function() { return gen.next(undefined); });
  });
}
```

意思和之前的***run***执行器类似。可以看到async是最简单的，最符合语义的，只要我们用到的所有一步方法都是返回的都是Promise，就能很轻松的使用async，Nodejs7已经支持async语法，只需添加***--harmony-async-await***启动参数即可。或者使用***[babel](https://babeljs.io/)***转码也行。