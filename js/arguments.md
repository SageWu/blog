# Arguments

`arguments`为类数组对象，可通过`index`访问元素以及拥有`length`属性

```js
//类数组转数组
// 1. slice
Array.prototype.slice.call(arrayLike);
// 2. splice
Array.prototype.splice.call(arrayLike, 0);
// 3. ES6 Array.from
Array.from(arrayLike);
// 4. concat
Array.prototype.concat.apply([], arrayLike)
```

* length

```js
function fun() {
    console.log(arguments.length);  //实参个数
}

console.log(fun.length);    //形参个数
```

* callee：被调用函数
* 在非严格模式下，arguments会和对应参数进行绑定

* leaking arguments

> Leaking the arguments object kills optimization because it forces V8 to instantiate the arguments as a Javascript object instead of optimizing them into stack variables.

```js
function leaksArguments1() {
    return arguments;
}

function leaksArguments2() {
    var args = [].slice.call(arguments);
}

function leaksArguments3() {
    var a = arguments;
    return function() {
        return a;
    };
}
```

* arrow function without arguments
当函数声明定义的时候，如果是箭头函数，会设置内部属性`[[ThisMode]]`为`lexical`，这样在函数执行的时候会去判断这个内部属性。
