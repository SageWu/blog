# 类

## 创建对象

1、工厂模式
缺点：无法区分类别，各自拥有方法的拷贝

```js
function factoryPerson(name) {
 var obj = new Object();
 obj.name = name;
 obj.say = function () {
  console.log(this.name);
 }

 return obj;
}

var person = factoryPerson("Bell");
```

2、构造函数模式
缺点：各自拥有方法的拷贝

```js
function Person(name) {
 this.name = name;
 this.say = function() {
  console.log(this.name);
 }
}

var person = new Person("Bell");
```

解决方案：

```js
var Person = (function() {
 function Person(name) {
  this.name = name;
  this.say = say;
 }

 function say() {
  console.log(this.name);
 }

 return Person;
}());

var person = new Person("Bell");
```

3、原型模式
缺点：所有属性和方法都是共享的，无法初始化

```js
var Person = (function() {
 function Person() {
 }

 Person.prototype = {
  constructor: Person,
  name: "Bell",
  say: function () {
   console.log(this.name);
  }
 }

 return Person;
}());

var person = new Person();
```

4、组合模式

```js
var Person = (function() {
 function Person(name) {
  this.name = name;
 }

 Person.prototype.say = function () {
  console.log(this.name);
 }

 return Person;
}());

var person = new Person("Bell");
```

5、其他
寄生构造函数模式，稳妥构造函数模式(类似闭包)

## 继承

1、经典继承(构造函数模式)
缺点：每个`child`实例都有父类的方法的拷贝

```js
function Parent(name) {
 this.name = name;
 this.say = function () {
   console.log(this.name);
 }
}

function Child(name) {
 Parent.call(this, name);
}
```

2、原型链继承(组合模式)
缺点：父类的属性和方法被共享，且实例化子类时不能初始化父类

```js
function Parent(name) {
 this.name = name;
}
Parent.prototype.say = function () {
 console.log(this.name);
}

function Child() {
}
Child.prototype = new Parent("bell");
Child.prototype.constructor = Child;
```

3、组合继承(组合模式)
缺点：实例对象和其原型对象重复拥有父类的属性

```js
function Parent(name) {
 this.name = name;
}
Parent.prototype.say = function () {
 console.log(this.name);
}

function Child(name) {
 Parent.call(this, name);
}
Child.prototype = new Parent("bell");
Child.prototype.constructor = Child;
```

4、寄生组合继承(组合模式)

```js
function Parent(name) {
 this.name = name;
}
Parent.prototype.say = function () {
 console.log(this.name);
}

function Child(name, age) {
 Parent.call(this, name);
 this.age = age;
}
function __() {
 this.constructor = Child;   
}
__.prototype = Parent.prototype;
Child.prototype = new __();
Child.prototype.print = function () {
 console.log(this.age);
}
```
