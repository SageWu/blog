# 执行上下文

## 作用域

词法作用域，即静态作用域，在函数声明的时候就已经确定。

## this

* undefined，非严格模式下为全局对象
* object
* call, apply, bind
* arrow function

箭头函数与普通函数的区别：

* 没有原型对象
* 使用new会报错
* 没有arguments
* 无法改变this指向

## 模拟实现

```js
Function.prototype.call = function (context) {
  context = context || window;
  context.__fn__ = this;

  var args = [];
  for(var i = 1, len = arguments.length; i < len; i++) {
    args.push('arguments[' + i + ']');
  }

  var result = eval('context.__fn__(' + args +')');
  delete context.__fn__;

  return result;
}
Function.prototype.call = function (context, ...args) {
  context = context || window;
  context.__fn__ = this;
  let result = context.__fn__(...args);
  delete context.__fn__;
  return result;
}

Function.prototype.apply = function (context, arr) {
  context = context || window;
  context.__fn__ = this;

  var result;
  if (!arr) {
    result = context.__fn__();
  }
  else {
    var args = [];
    for (var i = 0, len = arr.length; i < len; i++) {
      args.push('arr[' + i + ']');
    }
    result = eval('context.__fn__(' + args + ')');
  }

  delete context.__fn__;
  return result;
}
Function.prototype.apply = function (context, args) {
  context = context || window;
  context.__fn__ = this;
  let result = context.__fn__(...args);
  delete context.__fn__;
  return result;
}

Function.prototype.bind = function (context) {
  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - No function type");
  }

  var self = this;
  var bind_args = Array.prototype.slice.call(arguments, 1);
  var bind_fn = function () {
    var args = Array.prototype.slice.call(arguments);

    if(this.constructor.name === "Object") {
      var arg_strs = [];
      for(var i = 0; i < bind_args.length; i++) {
        arg_strs.push("bind_args[" + i + "]");
      }
      for(var i = 0; i < args.length; i++) {
        arg_strs.push("args[" + i + "]");
      }

      return eval("new self(" + arg_strs + ")");
    }
    else {
      return self.apply(context, bind_args.concat(args));
    }
  }
  bind_fn.prototype = undefined;

  return bind_fn;
}
Function.prototype.bind = function (context, ...bind_args) {
  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - No function type");
  }

  var self = this;
  var bind_fn = function (...args) {
    if(this.constructor.name === "Object")
      return new self(...bind_args, ...args);
    else
      return self.apply(context, bind_args.concat(args));
  }
  bind_fn.prototype = undefined;

  return bind_fn;
}

Object.prototype.create = function(o) {
  function f() {}
  f.prototype = o;
  return new f;
};

function objectFactory() {
  var constructor = [].shift.call(arguments);
  var obj = Object.create(constructor.prototype);
  var result = constructor.apply(obj, arguments);
  return result instanceof Object? result: obj;
};
```
