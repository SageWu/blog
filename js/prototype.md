# 原型链

* `prototype`: 构造函数的属性，指向原型对象
* `constructor`: 原型对象属性，指向构造函数
* `__proto__`: 任何对象都具有的属性(`内部属性为[[prototype]]，通过Object.prototype.__proto__的getter和setter进行访问和设置`)，指向原型对象。

在`new`对象的时候，如果构造函数的`prototype`属性为空，则`__proto__`指向`Object.prototype`。`Object.prototype.__proto__`为`null`，为原型链的终点。

![转载](../assets/images/prototype.png)

* 创建无原型对象

```js
var obj = Object.create(null);
console.log(obj.__proto__); //undefined

var obj = new Object();
obj.__proto__ = null
console.log(obj.__proto__); //undefined
```

> * 注：
> * 所有构造函数以及`Function.prototype`的类型均为`function`
> * `Function.__proto__ == Function.prototype //true`
> * `instanceof`为查找原型链是否包含指定原型对象
