# Async Hooks

## 基本概念

提供了用于跟踪异步资源的接口。\
异步资源指的是具有关联回调的对象。回调可被多次调用；资源可在回调调用前回收。

## 为何要跟踪异步资源

nodejs基于事件循环模型，当执行异步操作时，并不会阻塞主线程，而是在特定事件触发时，将回调添加到队列中排队执行，这时候调用方的上下文环境已经不存在。

* 无法确定调用链
* 异步回调发生异常，无法拿到完整的调用栈

## 解决方案

通过使用`async_hooks`，可以明确得到异步回调的调用链。

```js
const fs = require("fs")
const async_hooks = require("async_hooks")

const { fd } = process.stdout

async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId()
    fs.writeSync(
      fd,
      `${type}(${asyncId}): 触发方 => ${triggerAsyncId} 执行上下文 => ${eid}\n`
    )
  },
  before(asyncId) {
    fs.writeSync(
      fd,
      `开始异步回调 ${asyncId}\n`
    )
  },
  after(asyncId) {
    fs.writeSync(
      fd,
      `结束异步回调 ${asyncId}\n`
    )
  },
  destroy(asyncId) {
    fs.writeSync(
      fd,
      `销毁异步资源 ${asyncId}\n`
    )
  }
}).enable()


function callback() {
  fs.writeSync(fd, "写入成功\n")
}

fs.writeSync(fd, "开始写 a.txt\n")
fs.writeFile("a.txt", "a", callback)
fs.writeSync(fd, "开始写 b.txt\n")
fs.writeFile("b.txt", "b", callback)


开始写 a.txt
FSREQCALLBACK(4): 触发方 => 1 执行上下文 => 1
开始写 b.txt
FSREQCALLBACK(5): 触发方 => 1 执行上下文 => 1
开始异步回调 4
FSREQCALLBACK(6): 触发方 => 4 执行上下文 => 4
结束异步回调 4
开始异步回调 5
FSREQCALLBACK(7): 触发方 => 5 执行上下文 => 5
结束异步回调 5
销毁异步资源 4
销毁异步资源 5
开始异步回调 6
FSREQCALLBACK(8): 触发方 => 6 执行上下文 => 6
结束异步回调 6
开始异步回调 7
FSREQCALLBACK(9): 触发方 => 7 执行上下文 => 7
结束异步回调 7
销毁异步资源 6
销毁异步资源 7
开始异步回调 8
写入成功
结束异步回调 8
开始异步回调 9
写入成功
结束异步回调 9
销毁异步资源 8
销毁异步资源 9
```

通过使用`async_hooks`+`Error.captureStackTrace`，异步回调发生异常时，可以得到完整的调用栈。

```js
const async_hooks = require("async_hooks")

const asyncResource = new Map()

async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const stackTrace = {}
    Error.captureStackTrace(stackTrace, arguments.callee)

    asyncResource.set(asyncId, {
      asyncId,
      type,
      triggerAsyncId,
      stack: stackTrace.stack
    })
  },
  destroy(asyncId) {
    asyncResource.delete(asyncId)
  }
}).enable()

function main() {
  setTimeout(() => {
    throw new Error("错误消息")
  })
}
main()

function logStackTrace(asyncId) {
  const r = asyncResource.get(asyncId)
  if (!r) {
    return
  } else {
    console.log(`${r.type}(${r.asyncId})\n${r.stack}\n`)
    if (r.triggerAsyncId) {
      logStackTrace(r.triggerAsyncId)
    }
  }
}

process.on("uncaughtException", () => {
  logStackTrace(async_hooks.executionAsyncId())
})

```

## 性能

使用`async_hooks`对性能是有一定影响的，可参照`https://github.com/bmeurer/async-hooks-performance-impact`。

## 自定义异步资源

## AsyncLocalStorage

用于在异步调用链中存储状态，在异步持续时间内一直存在。

```js
const http = require("http")
const { AsyncLocalStorage } = require("async_hooks")

const asyncLocalStorage = new AsyncLocalStorage()

function logWithId(msg) {
  const id = asyncLocalStorage.getStore()
  console.log(id, msg)
}

let id = 0
http.createServer((req, res) => {
  asyncLocalStorage.run(id++, () => {
    logWithId("接收请求")
    setImmediate(() => {
      logWithId("结束请求")
      res.end()
    })
  })
}).listen(8080)

http.get('http://localhost:8080')
http.get('http://localhost:8080')


// 输出
0 接收请求
1 接收请求
0 结束请求
1 结束请求
```

实现原理：`https://github.com/nodejs/node/blob/master/lib/async_hooks.js#L267`。
