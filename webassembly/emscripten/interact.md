# 交互

## C++

### 直接暴露

`EMSCRIPTEN_KEEPALIVE`\
导出函数，和将函数添加到`EXPORTED_FUNCTIONS`参数一样，防止LLVM dead-code-eliminate该函数。\
之后通过`Module._[function name]`访问。

```cpp
int EMSCRIPTEN_KEEPALIVE add(int a, int b) {
  return a + b;
}
```

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

`emscripten_run_script`通过`eval`执行JS代码。\
`emscripten_sync_run_in_main_runtime_thread`可指定在主线程上执行。

`EM_JS`

```cpp
EM_JS(void, name, (), {
  //...
});

// 字符串
EM_JS(void, say_hello, (const char* str), {
  console.log('hello ' + UTF8ToString(str));
}

// 返回字符串
EM_JS(char*, get_unicode_str, (), {
  var jsString = 'Hello with some exotic Unicode characters: Tässä on yksi lumiukko: ☃, ole hyvä.';
  // 'jsString.length' would return the length of the string as UTF-16
  // units, but Emscripten C strings operate as UTF-8.
  var lengthBytes = lengthBytesUTF8(jsString)+1;
  var stringOnWasmHeap = _malloc(lengthBytes);
  stringToUTF8(jsString, stringOnWasmHeap, lengthBytes);
  return stringOnWasmHeap;
});

int main() {
  char* str = get_unicode_str();
  printf("UTF8 string says: %s\n", str);
  // Each call to _malloc() must be paired with free(), or heap memory will leak!
  free(str);
  return 0;
}

// 指针
EM_JS(void, read_data, (int* data), {
  console.log('Data: ' + HEAP32[data>>2] + ', ' + HEAP32[(data+4)>>2]);
});

int main() {
  int arr[2] = { 30, 45 };
  read_data(arr);
  return 0;
}
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
