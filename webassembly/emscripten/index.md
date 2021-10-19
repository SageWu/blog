# emscripten

## 介绍

另外提供了SDL、OpenGL的浏览器支持。

## 安装

```bash
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk
./emsdk install latest
./emsdk activate latest

添加环境变量，不建议添加到全局，由于emsdk使用独立的nodejs和python版本。
source ./emsdk_env.sh 
```

## 编译

常规方式

```bash
emcc/em++ --bind -o a.html a.cpp
```

cmake方式

```bash
emcmake cmake
emmake make
```

## 调试

获取编译时调试信息，设置 `EMCC_DEBUG`。\
其他帮助调试参数：`ASSERTIONS`, `SAFE_HEAP`, `DEMANGLE_SUPPORT`, `EMCC_AUTODEBUG`。

## 交互

### C++方法

`EMSCRIPTEN_KEEPALIVE`\
导出函数，类似将函数添加到`EXPORTED_FUNCTIONS`编译参数。同时防止LLVM dead-code-eliminate该函数。\
之后通过`Module._[function name]`访问。

```cpp
void EMSCRIPTEN_KEEPALIVE add(float a, float b) {
  return a + b;
}
```

### 执行JS

#### `emscripten_run_script`

直接执行JS代码字符串，但存在一定性能问题。\
如果在pthread上执行，则该代码在属于这个线程的WebWorker上运行。
> emscripten_sync_run_in_main_runtime_thread

#### `EM_JS`

EM_JS(return_type, function_name, arguments, code)

```cpp
// 数字
EM_JS(void, take_args, (int x, float y), {
  console.log('I received: ' + [x, y]);
});

int main() {
  take_args(100, 35.5);
  return 0;
}

// 字符串
EM_JS(void, say_hello, (const char* str), {
  console.log('hello ' + UTF8ToString(str));
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
```

#### `EM_ASM`

EM_JS(code, ...arguments)，直接执行JS代码。参数通过`$0`等访问。

### 自定义JS方法

```cpp
extern "C" {
  extern void jsMethod();
}
```

```js
mergeInto(LibraryManager.library, {
  jsMethod: function() {
    console.log("call from library js");
  },
});
```

```bash
emcc --js-library library.js
```

## 多线程支持

编译时，添加`-s PROXY_TO_PTHREAD=1`参数，会自动添加创建线程的逻辑，并在线程上执行`main`函数的逻辑。\
在浏览器中，通过`WebWorker`实现`pthread`支持，主线程与其它`WebWorker`线程通过`SharedArrayBuffer`实现原子操作以及通信。
