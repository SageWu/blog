# Component

## 类组件

继承`React.Component`。构造函数初始化`props`、`context`，`refs`设置为空对象，`updater`设置为空的更新器，之后会注入真正的更新器。

### Component.prototype.isReactComponent

默认值为空对象

### Component.prototype.setState

> 调用`updater.enqueueSetState`，入队更新状态操作。
>> 1、获取`Fiber`节点，通过类组件实例的`_reactInternals`属性获取。
>> 2、`requestEventTime`，请求当前时间。如果当前在`render`或者`commit`阶段，直接返回`now()`；否则使用缓存的`currentEventTime`，使得在浏览器事件阶段的所有更新具有相同的事件时间。
>> 3、`requestUpdateLane`，获取更新所属的车道。如果非并发模式，则直接返回`SyncLane`。如果当前在`render`阶段，且不推迟i到下一个批次，则从本次渲染的车道中随机选择一个车道。如当前批量配置中，有配置transition，则从`TransitionLanes`中依次选择一个，并保持当前浏览器事件的所有更新具有相同的过渡车道。如果为执行React方法内部触发的更新，根据当前更新优先级选择车道。宿主环境根据事件类型，选择车道。
>> 4、`createUpdate`，创建一个更新对象，包含了`eventTime`、`lane`、`tag`、`payload`、`callback`。
>> 5、`enqueueUpdate`，将更新入队到对于Fiber的更新队列中。更新队列为环状链表，`pending`或`interleaved`指向最后一个更新。如果在并发模式且处于可中断渲染过程中，则将更新推送到`interleaved`，在可中断渲染结束时，会移到`pending`队列中。否则推送到`pending`。
>> 6、`scheduleUpdateOnFiber`
>>> 1、检查嵌套更新是否超限制
>>> 2、`markUpdateLaneFromFiberToRoot`，更新当前Fiber的`lanes`，并一路往上，更新父Fiber节点的`childLanes`。
>>> 3、`markRootUpdated`，更新`FiberRoot`的`pendingLanes`，如果本次更新选择的车道非`IdleLane`，则清除`suspendedLanes`和`pingedLanes`，尝试重新处理这些车道的更新。更新`eventTimes`。
>>> 4、如果当前处于可中断渲染过程，则更新`workInProgressRootUpdatedLanes`。
>>> 5、`ensureRootIsScheduled`
>>>> 1、遍历`pendingLanes`，如果不存在过期时间，为非挂起的或者是挂起但`pinged`的车道计算过期时间，因为这些车道为CPU相关任务。如果过期时间已经到达，则更新`expiredLanes`。
>>>> 2、`getNextLanes`，如果不存在需要处理的车道，则做清除操作。如果和上次调度任务优先级一致，则返回。清除之前的调度任务，重新调度。
>>>> 3、新调度任务优先级，如果为`SyncLane`，则在内部同步队列（非`Scheduler`）中调度同步任务`performSyncWorkOnRoot`，紧接着调度`flushSyncCallbacks`，用于清空内部同步队列。如果为其他优先级，则调度不同优先级的`performConcurrentWorkOnRoot`。最后保存调度任务相关信息。
>>>
>>> 6、如果非并发模式且当前执行上下文`executionContext`为空，则马上清空内部同步队列。

注意：由于在某些异步环境下，`executionContext`为空，多次更新状态造成多次渲染，可使用`unstable_batchedUpdates`，合并多次状态更新。

### Component.prototype.forceUpdate

> 调用`updater.enqueueForceUpdate`，入队强制更新操作。
> 后续流程和`setState`一致，唯一不同点是更新的`tag`为`ForceUpdate`。

`isMounted`和`replaceState`已经被废弃。

问题：
> 类中表达式的方法与`prototype`上设置的方法优先级？
> 类中表达式方法优先级更高，由于直接设置在实例对象上。另外声明式的方法，是设置在原型上的。

旧版React，优先级：ReactDom.flushSync > 同步批量更新 > 异步更新

## 函数组件

### 与类组件本质区别

类组件底层只需要实例化一次，保存相应的状态，更新时只需重新执行`render`和执行对应的生命周期函数。而函数组件，每一次更新都是一次新的函数调用。\
因此函数组件引入`hooks`，保存状态以及执行副作用钩子。

## 组件通信

### props, callback

父组件通过修改`state`，重新渲染，传递`props`给子组件。\
子组件通过调用`props`中的回调，通知父组件。

### ref

### react-redux等状态管理器

### context

### event bus

比较适用于用React做基础构建的小程序，如Taro。\
虽然该方式可以跨组件层级，但存在缺点：需要手动绑定和解绑；状态流未知，难以维护；违背React数据流向原则。

## HOC高阶组件
