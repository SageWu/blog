# 异步

`Asyncify`使得同步C/C++代码和异步JS交互。通过`-s ASYNCIFY`开启。

`emscripten_sleep`\
emscripten会暂停C/C++，将执行权转交给EventLoop，之后再恢复执行。

`EM_ASYNC_JS`\
emscripten会暂停C/C++，等待JS异步操作完成，之后再恢复执行。\
如果JS引擎不支持`async/await`，则可以使用`EM_JS`和`Asyncify.handleAsync`，并在构建时设置`ASYNCIFY_IMPORTS`。\
如果JS引擎不支持`Promise`，则可以使用`Asyncify.handleSleep`。
