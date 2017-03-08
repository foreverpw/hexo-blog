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

##### 5、Cookie的弊端
>1.IE6或更低版本最多20个cookie,IE7和之后的版本最后可以有50个cookie,Firefox最多50个cookie,每个cookie长度大约不能超过4KB(各个浏览器有微小差异，大约都是4KB)
>
>2.安全性问题。如果cookie被人拦截了，那人就可以取得所有的session信息。即使加密也与事无补，因为拦截者并不需要知道cookie的意义，他只要原样转发cookie就可以达到目的了。

##### 6、display:none和visibility:hidden的区别
>display:none  隐藏对应的元素，在文档布局中不再给它分配空间，它各边的元素会合拢，就当他从来不存在。
>
>visibility:hidden  隐藏对应的元素，但是在文档布局中仍保留原来的空间。

##### 7、CSS中 link 和@import 的区别
1. link属于HTML标签，而@import是CSS提供的
2. 页面被加载的时，link会同时被加载载，而@import引用的CSS会等到页面被加载完再加载;
3. import只在IE5以上才能识别，而link是HTML标签，无兼容问题
4. link方式的样式的权重 高于@import的权重

##### 8、new操作符具体干了什么
```javascript
var obj  = {};
obj.__proto__ = Base.prototype;
Base.call(obj);
```

##### 9、http状态码有那些,分别代表是什么意思
>100-199 用于指定客户端应相应的某些动作。
>
>200-299 用于表示请求成功。
>
>300-399 用于已经移动的文件并且常被包含在定位头信息中指定新的地址信息。
>
>400-499 用于指出客户端的错误。400    1、语义有误，当前请求无法被服务器理解。401   当前请求需要用户验证 403  服务器已经理解请求，但是拒绝执行它。
>
>500-599 用于支持服务器错误。 503 – 服务不可用