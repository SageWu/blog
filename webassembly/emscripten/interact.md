# 交互

## C++

### 直接调用

`cwrap`对一个C++函数进行包裹，使得可以直接在JS中调用。值的类型有null、number、string、array。

```js
let add = Module.cwrap("add", "number", ["number", "number"]);
let res = add(1, 2);
```

`ccall`直接调用。

```js
let res = Module.ccall("add", "number", ["number", "number"], [1, 2]);
```

### 内联JS

`emscripten_run_script`通过`eval`执行JS代码。

`EM_JS`

```cpp
EM_JS(void, name, (), {
  //...
});
```

`EM_ASM`

```cpp
EM_ASM(
  alert('txt');
);

int res = EM_ASM_INT({
  return $0;
}, 1);
```

## JS

### 实现C函数

所有JS库的代码存放在`LibraryManager.library`上，使用`mergeInto`添加新的API。

```js
mergeInto(LibraryManager.library, {
  alert: function() {
    alert('hello world');
  },
});
```

函数会被`toString`再使用。因此类似闭包则会产生错误。

```js
mergeInto(LibraryManager.library, {
  // 引用其他函数
  fn1__postset: '_fn1();',
  fn1: function() {
    _fn1 = document.querySelector.bind(document);
  },
  // 闭包
  fn2__postset: '_fn2();',
  fn2: function() {
    var count = 0;
    _fn2 = function() {
      console.log("times called: ", ++count);
    };
  },
});
```

> 在emscripten中，方法会以`_`（下划线）为前缀。以`$`开头则最终会去掉。\
> __deps、autoAddDeps(myLibrary, name)

### 函数指针

通过使用`addFunction`，获取JS函数的指针，传递给C/C++调用。\
如果使用LLVM wasm backend，则需传递第二个参数即函数签名，第一个字母为返回值类型，其他为参数类型。

* `v`: void type
* `i`: 32-bit integer type
* `j`: 64-bit integer type (currently does not exist in JavaScript)
* `f`: 32-bit float type
* `d`: 64-bit float type
