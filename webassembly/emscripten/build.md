# 构建

## 命令

`emcmake cmake`\
根据CMakeLists.txt生成makefile。

`emconfigure ./configure`\
将makefile中gcc替换为emcc，并设置合适的环境变量，像CXX、CC。

`emmake make`\
根据makefile构建项目。\
可能构建后的产物为`.a（静态库）、.so（动态库）、.o（对象文件）`，可通过`emcc`构建为JS+WASM。

> wasm对象文件以`\0asm`开头，LLVM bitcode以`BC`开头。

## 产物

`-o *.html`\
html + js + wasm

`-o *.js`\
js + wasm

`-o *.wasm`\
wasm。和设置`STANDALONE_WASM`一样。

* --emit-symbol-map
* -gsource-map
* --memory-init-file 1
* --preload-file xxx

## 优化

主要分两个阶段。LLVM在构建源文件的时候优化；JavaScript/WebAssembly特定的优化。

### -O

优化编译产物。\
0: 无优化，构建速度快。\
1: 简单优化。\
2: 启动更多优化，更多的JS优化。\
3: 适合于发布。\
s: 更关注产物大小，与运行速度进行权衡。\
z: 更进一步减少产物大小。

### --closure

使用closure编译器优化js代码体积。\
0: 不使用closure编译器，-O2及以下默认设置该参数。\
1: 减少js体积，除了wasm和asm.js。\
2: 同时也减少asm.js体积，但也导致了部分优化失效。

## 参数

* --pre-js: 添加js代码到emscripten输出的js代码前面，一起进行优化。
* --post-js: 和pre-js类似，但是位置为后面。
* --js-library: 链接除核心库以外的js库。
* --cpuprofiler: 页面嵌入cpu分析器。
* --memoryprofiler: 页面嵌入内存分配跟踪器。
* --threadprofiler: 页面嵌入线程活动分析器。
* -g: 保留调试信息。
* -gseparate-dwarf=[filename]: 将dwarf调试信息输出到独立文件。
* -s SEPARATE_DWARF_URL=[url]: 自定义调试信息文件地址。
* --source-map-base: wasm的sourcemap存放地址。
* -fexceptions: 基于JS的异常处理。
* -fwasm-exceptions: 原生wasm异常处理。
* -lopenal: 链接openal。
* -lidbfs.js: 开启idb文件系统。
* -lworkerfs.js: 开启worker文件系统。
* -flto: 启动链接优化。

## flags

* `WASM`: 是否输出wasm模块文件。在某些不支持wasm情况下，只输出js代码，但性能欠缺。
* `ENVIRONMENT`: 支持的环境。
* `EXPORTED_RUNTIME_METHODS`: 需要导出的运行时元素，附加在Module上。
* `FORCE_FILESYSTEM`: 强制包含文件系统。

* `USE_PTHREADS`: 支持pthread。
* `PTHREAD_POOL_SIZE`: 线程池大小。由于worker的初始化是异步的，如果想要同步创建worker，则可以预先创建worker池。
* `FULL_ES2`: 支持所有GLES2特性，不只是WebGL。
* `FULL_ES3`: 支持所有GLES3特性。
* `GL_DEBUG`: 更多WebGL调试信息。
* `FETCH`: 启动emscripten_fetch api。
* `OFFSCREENCANVAS_SUPPORT`: 支持离屏canvas，转移canvas到pthread并且创建WebGL上下文。
* `OFFSCREENCANVASES_TO_PTHREAD`: 代理指定canvas到pthread。

* `DISABLE_EXCEPTION_CATCHING`: 禁用C++异常捕获。
* `EXCEPTION_CATCHING_ALLOWED`: 只允许从哪里捕获异常。
* `WARN_ON_UNDEFINED_SYMBOLS`: 当符号无法解析时，输出警告。

* `ASSERTIONS`: 运行时断言。
* `ALLOW_MEMORY_GROWTH`: 允许内存增长。
* `ALLOW_TABLE_GROWTH`: 允许运行时添加更多函数到表格。
* `SAFE_HEAP`: 每次写入堆时检查。

* `MODULARIZE`: 模块化。导出工厂函数用于实例化模块。
* `EXPORT_NAME`: 导出名称。
* `EXPORT_ES6`: 使用es6模块化规范。

## profile

```bash
export EMPROFILE=1
...
emprofile
```
