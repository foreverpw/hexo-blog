layout: draf
title: javascript中的数字
tags:
  - js
date: 2017-03-03 15:23:07
---

javascript中的数字其实都是浮点数表示的，toFixed，小数相加，整数相加都会丢失精度：
> 9007199254740992 + 1 = 9007199254740992
>
> 9007199254740992 + 2 = 9007199254740994

#### IEEE浮点表示法
javascript使用的是IEEE 754 64位的浮点表示法，格式如下：

| s | e | f |
| :--: | :------: | :------: |
| 1bit |  11 bit  |  52bit   |

浮点数表示标准形式：V = (-1)^s×M×2^E，拆开各种情况公式如下：

| (−1)^*s* × %1.*f* × 2^(*e*−(2^10-1)) | normalized, 0 < *e* < 2^11-1   |
| ------------------------------------ | ------------------------------ |
| (−1)^*s* × %0.*f* × 2^(*e*−(2^10-1)) | denormalized, *e* = 0, *f* > 0 |
| (−1)^*s* × 0                         | *e* = 0, *f* = 0               |
| NaN                                  | *e* = 2^11-1, *f* > 0          |
| infinity                             | *e* = 2^11-1, *f* = 0          |

M = 1.f 或 0.f，所以精度就由f决定，当把10进制数字转成浮点数时，首先要转成**1.f**或**0.f**的二进制结构，当转换成上面结构后的二进制数字精度大于（52+1）位时就会产生精度丢失的情况（不够表示了）

#### 解决方案
先升幂再降幂：
```JavaScript
function add(num1, num2){
  let r1, r2, m;
  r1 = (''+num1).split('.')[1].length;
  r2 = (''+num2).split('.')[1].length;

  m = Math.pow(10,Math.max(r1,r2));
  return (num1 * m + num2 * m) / m;
}
```
toFixed修复：
```javascript
// toFixed 修复
function toFixed(num, s) {
    var times = Math.pow(10, s)
    var des = num * times + 0.5
    des = parseInt(des, 10) / times
    return des + ''
}
```