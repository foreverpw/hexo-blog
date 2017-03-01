---
title: 记不住的css
date: 2017-02-28 17:49:54
tags:
- css
---
##### 1、让页面里的字体变清晰，变细
```css
-webkit-font-smoothing: antialiased;
```

##### 2、如何修改chrome记住密码后自动填充表单的黄色背景
```css
input:-webkit-autofill, textarea:-webkit-autofill, select:-webkit-autofill {
  background-color: rgb(250, 255, 189); /* #FAFFBD; */
  background-image: none;
  color: rgb(0, 0, 0);
}
```

##### 3、标题隐藏，超出部分显示省略号
```css
overflow:hidden; text-overflow: ellipsis;white-space: nowrap;
```

##### 4、css定义的权重
```css
/*权重为1*/
div{
}
/*权重为10*/
.class1{
}
/*权重为100*/
#id1{
}
/*权重为100+1=101*/
#id1 div{
}
/*权重为10+1=11*/
.class1 div{
}
/*权重为10+10+1=21*/
.class1 .class2 div{
}
```

##### 5、CSS选择符有哪些？哪些属性可以继承？
    1.id选择器（ # myid）
    2.类选择器（.myclassname）
    3.标签选择器（div, h1, p）
    4.相邻选择器（h1 + p）
    5.子选择器（ul > li）
    6.后代选择器（li a）
    7.通配符选择器（ * ）
    8.属性选择器（a[rel = "external"]）
    9.伪类选择器（a:hover, li:nth-child）
    
* 可继承的样式： font-size font-family color, UL LI DL DD DT;
* 不可继承的样式：border padding margin width height ;

##### 6、简单的文字模糊效果
```css
p {
    color: transparent;
    text-shadow: #111 0 0 5px;
}
```

##### 7、calc运算
```css
.container{
    background-position: calc(100% - 50px) calc(100% - 20px);
}
```

##### 8、利用重复指定box-shadow来达到多个边框的效果
```css
div {
    box-shadow: 0 0 0 6px rgba(0, 0, 0, 0.2), 0 0 0 12px rgba(0, 0, 0, 0.2), 0 0 0 18px rgba(0, 0, 0, 0.2), 0 0 0 24px rgba(0, 0, 0, 0.2);
    height: 200px;
    margin: 50px auto;
    width: 400px
}
```

##### 9、实时编辑CSS
通过设置style标签的display:block样式可以让页面的style标签显示出来，并且加上contentEditable属性后可以让样式成为可编辑状态，更改后的样式效果也是实时更新呈现的。此技巧在IE下无效。
```html
<!DOCTYPE html>
<html>
    <body>
        <style style="display:block" contentEditable>
            body { color: blue }
        </style>
    </body>
</html>
```
