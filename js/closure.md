# 闭包

## 介绍

* 广义：能够访问自由变量(非局部变量)的函数
* 狭义：创建函数的执行上下文已经销毁，存在对自由变量的引用，故`AO`并没有被销毁

```js
var data = [];
for (var i = 0; i < 3; i++) {
  data[i] = function () {
    console.log(i);
  };
}

data[0]();data[1]();data[2]();

//形成闭包
var data = [];
for (var i = 0; i < 3; i++) {
  data[i] = (function (i) {
    return function(){
      console.log(i);
    }
  })(i);
}

data[0]();data[1]();data[2]();
```

`let`产生块级作用域，并且使每次循环变成新的作用域。

```ts
let data = [];
for (let i = 0; i < 3; i++) {
  data[i] = function () {
    console.log(i);
  };
}
```

## 应用场景

* 保护变量安全
* 缓存变量
* 避免污染全局作用域
* 实现模块
