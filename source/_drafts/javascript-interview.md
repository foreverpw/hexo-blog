---
title: 前端面试题
tags:
- 面试
categories:
- 前端
---
##### 1、使用 typeof bar === "object" 判断 bar 是不是一个对象有神马潜在的弊端？如何避免这种弊端？
使用 typeof 的弊端是显而易见的(这种弊端同使用 instanceof)：

```javascript
let obj = {};
let arr = [];

console.log(typeof obj === 'object');  //true
console.log(typeof arr === 'object');  //true
console.log(typeof null === 'object');  //true
```

从上面的输出结果可知，typeof bar === "object" 并不能准确判断 bar 就是一个 Object。可以通过 Object.prototype.toString.call(bar) === "[object Object]" 来避免这种弊端：

```javascript
let obj = {};
let arr = [];

console.log(Object.prototype.toString.call(obj));  //[object Object]
console.log(Object.prototype.toString.call(arr));  //[object Array]
console.log(Object.prototype.toString.call(null));  //[object Null]
```

##### 2、写出以下代码的输出结果
```javascript
function Foo() {
    getName = function () { alert (1); };
    return this;
}
Foo.getName = function () { alert (2);};
Foo.prototype.getName = function () { alert (3);};
var getName = function () { alert (4);};
function getName() { alert (5);}

Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
new new Foo().getName();
```

```javascript
//答案：
Foo.getName();//2
getName();//4
Foo().getName();//1
getName();//1
new Foo.getName();//2
new Foo().getName();//3
new new Foo().getName();//3
```

##### 3、AMD、CMD、Commonjs的区别
参考玉伯的[回答](https://www.zhihu.com/question/20351507/answer/14859415)

AMD和CMD都是提前读取（下载）了文件，但AMD是提前加载（执行）文件中的代码，CMD是按需加载（执行）文件中的代码。
Commonjs 是同步（阻塞式）加载模块。

##### 4、如果 list 很大，下面的这段递归代码会造成堆栈溢出。如果在不改变递归模式的前提下修善这段代码？
```javascript
var list = readHugeList();

var nextListItem = function() {
    var item = list.pop();

    if (item) {
        // process the list item...
        nextListItem();
    }
};
```

通过setTimeout异步执行,这样堆栈会及时被清理
```javascript
var list = readHugeList();

var nextListItem = function() {
    var item = list.pop();

    if (item) {
        // process the list item...
        setTimeout( nextListItem, 0);
    }
};
```

##### 5、