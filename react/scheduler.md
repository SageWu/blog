# 调度器

## 核心

使用最小堆数据结构，存储格式为数组。优先级比较为：`sortIndex`, `id`。\
向宿主请求执行权方式：旧ie和nodejs环境下，使用`setImmediate`；浏览器和Worker环境下，使用`MessageChannel`；其他环境下，使用`setTimeout`。

## scheduleCallback

调度任务
> `getCurrentTime`，获取当前时间，使用`performance.now`或`Date.now`。
> 计算开始时间，可以配置延时。根据优先级，定义超时时间，`IdelPriority`永不超时。根据开始时间和超时时间计算过期时间。
> 创建任务，包含`priorityLevel`, `callback`, `startTime`, `expirationTime`, `sortIndex`。
> 如果开始时间等于当前时间，`sortIndex`更新为`expirationTime`，再将任务加入最小堆。
> 如果当前不在处理任务并且宿主回调还没调度，则请求宿主回调`flushWork`。

## performWorkUntilDeadline

> 计算`deadline`。
> 执行宿主回调（`flushWork`）。
> 如果还有更多任务，则再次请求宿主回调。
