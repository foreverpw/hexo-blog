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

