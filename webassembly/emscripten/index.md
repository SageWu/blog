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

## 多线程支持

编译时，添加`-s PROXY_TO_PTHREAD=1`参数，会自动添加创建线程的逻辑，并在线程上执行`main`函数的逻辑。\
在浏览器中，通过`WebWorker`实现`pthread`支持，主线程与其它`WebWorker`线程通过`SharedArrayBuffer`实现原子操作以及通信。
