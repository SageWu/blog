# Component

## 类组件

继承`React.Component`。构造函数初始化`props`、`context`，`refs`设置为空对象，`updater`设置为空的更新器，之后会注入真正的更新器。

### Component.prototype.isReactComponent

默认值为空对象

### Component.prototype.setState

> 调用`updater.enqueueSetState`，入队更新状态操作。
>> 1、获取`Fiber`节点，通过类组件实例的`_reactInternals`属性获取。\
>> 2、`requestEventTime`，请求当前时间。如果当前在`render`或者`commit`阶段，直接返回`now()`；否则使用缓存的`currentEventTime`，使得在浏览器事件阶段的所有更新具有相同的事件时间。\
>> 3、`requestUpdateLane`，获取更新所属的车道。如果非并发模式，则直接返回`SyncLane`。如果当前在`render`阶段，且不推迟到下一个批次，则从本次渲染的车道中随机选择一个车道。如当前批量配置中，有配置transition，则从`TransitionLanes`中依次选择一个，并保持当前浏览器事件的所有更新具有相同的过渡车道。如果为执行React方法内部触发的更新，根据当前更新优先级选择车道。宿主环境根据事件类型，选择车道。\
>> 4、`createUpdate`，创建一个更新对象，包含了`eventTime`、`lane`、`tag`、`payload`、`callback`。\
>> 5、`enqueueUpdate`，将更新入队到对于Fiber的更新队列中。更新队列为环状链表，`pending`或`interleaved`指向最后一个更新。如果在并发模式且处于可中断渲染过程中，则将更新推送到`interleaved`，在可中断渲染结束时，会移到`pending`队列中。否则推送到`pending`。\
>> 6、`scheduleUpdateOnFiber`
>>> 1、检查嵌套更新是否超限制\
>>> 2、`markUpdateLaneFromFiberToRoot`，更新当前Fiber的`lanes`，并一路往上，更新父Fiber节点的`childLanes`。\
>>> 3、`markRootUpdated`，更新`FiberRoot`的`pendingLanes`，如果本次更新选择的车道非`IdleLane`，则清除`suspendedLanes`和`pingedLanes`，尝试重新处理这些车道的更新。更新`eventTimes`。\
>>> 4、如果当前处于可中断渲染过程，则更新`workInProgressRootUpdatedLanes`。\
>>> 5、`ensureRootIsScheduled`
>>>> 1、遍历`pendingLanes`，如果不存在过期时间，为非挂起的或者是挂起但`pinged`的车道计算过期时间，因为这些车道为CPU相关任务。如果过期时间已经到达，则更新`expiredLanes`。\
>>>> 2、`getNextLanes`，如果不存在需要处理的车道，则做清除操作。如果和上次调度任务优先级一致，则返回。清除之前的调度任务，重新调度。\
>>>> 3、新调度任务优先级，如果为`SyncLane`，则在内部同步队列（非`Scheduler`）中调度同步任务`performSyncWorkOnRoot`，紧接着调度`flushSyncCallbacks`，用于清空内部同步队列。如果为其他优先级，则调度不同优先级的`performConcurrentWorkOnRoot`。最后保存调度任务相关信息。
>>>
>>> 6、如果非并发模式且当前执行上下文`executionContext`为空，则马上清空内部同步队列。

注意：由于在某些异步环境下，`executionContext`为空，多次更新状态造成多次渲染，可使用`unstable_batchedUpdates`，合并多次状态更新。

### Component.prototype.forceUpdate

> 调用`updater.enqueueForceUpdate`，入队强制更新操作。\
> 后续流程和`setState`一致，唯一不同点是更新的`tag`为`ForceUpdate`。

注意：该更新用于标记是否有强制更新，从而使跳过更新失效。

`isMounted`和`replaceState`已经被废弃。

问题：
> 类中表达式的方法与`prototype`上设置的方法优先级？\
> 类中表达式方法优先级更高，由于直接设置在实例对象上。另外声明式的方法，是设置在原型上的。
>
> 旧版React，优先级？\
> ReactDom.flushSync > 同步批量更新 > 异步更新

### 生命周期函数

1、`componentWillReceiveProps`\
触发条件：没有实现新的生命周期函数，`props`或者`context`发生变化。

2、`getDerivedStateFromProps`\
触发条件：前面处理完`updateQueue`后，跳过更新条件不成立，条件为`props`, `context`和`state`没发生变化以及没有强制更新。

3、`shouldComponentUpdate`\
如果没有强制更新，则检查是否需要更新。如果有实现该函数则执行，如果是`PureComponent`的话，则浅比较`props`和`state`。

如果需要更新

4、`componentWillUpdate`\
触发条件：需要更新，并且没有实现新的生命周期函数。

5、`getSnapshotBeforeUpdate`\
触发条件：需要更新。

6、`componentDidUpdate`\
触发条件：需要更新。

否则

如果有更新已经合并到`current`上，则需触发`gSBU`和`cDU`。

## 函数组件

### 与类组件本质区别

类组件底层只需要实例化一次，保存相应的状态，更新时只需重新执行`render`和执行对应的生命周期函数。而函数组件，每一次更新都是一次新的函数调用。\
由于`this`是可变的，所以通常情况下，类组件获取到的是最新的`props`和`state`;而函数组件则会捕获渲染时所用的`props`和`state`，故而使用的是特定渲染的值。

* 代码更加简洁
* 逻辑关注点分离
* 逻辑抽离不会造成嵌套（HOC、render props）和props冲突（HOC）
* 没有生命周期函数，只有副作用钩子

### hooks

#### renderWithHooks

> 清空`wrokInProgress`的`memorizedState`, `updateQueue`, `lanes`。\
> 更新`ReactCurrentDispatcher`，即React的分发器，挂载阶段（current或current.memorizedState为空）和更新阶段的分发器是不同的。\
> 执行函数组件，获得子元素（ReactElements）。\
> 更新`ReactCurrentDispatcher`为`ContextOnlyDispatcher`。

#### useReducer

挂载阶段
> 创建hook，并加入到Fiber节点的`memorizedState`，以链表的形式存在。\
> 更新初始化状态到hook的`memorizedState`和`baseState`。\
> 初始化hook的`queue`，包含`pending`, `interleaved`, `lanes`, `dispatch`, `lastRenderedReducer`。\
> 更新`dispatch`为分发动作函数，绑定了`queue`，用于后续新增更新。\
> 返回`memorizedState`以及`dispatch`。

更新阶段
> 获取hook，如果workInProgress存在hook的话，则复用，否则从current克隆过来。\
> 如果hook上存在pending队列的话，将其移动到currentHook的`baseQueue`上，为环状链表。\
> 遍历currentHook上的`baseQueue`。当遇到第一个优先级不满足本次渲染过程的更新时，保存当前状态为`baseState`，包括该更新以及后续的更新都会添加到之后的`baseQueue`中。更新所属的车道处于本次渲染过程中，会进行处理，并更新`memorizedState`。处理更新的过程为，如果reducer没发生改变的话，则可以复用之前的计算结果，否则需重新计算。\
> 跳过的更新以及interleaved队列中的更新其所属的车道会更新到workInProgress的lanes上，也标记为跳过。\
> 返回最新的`memorizedState`和`dispatch`。

#### useState

基本上和`useReducer`一致，唯一区别在于使用内置的`reducer`。

#### useMemo

挂载阶段
> 创建hook。\
> 计算结果，并将结果和依赖保存到hook的`memorizedState`中。\
> 返回计算结果。

更新阶段
> 获取hook。\
> 如果新的依赖不为空，且和之前的依赖一致，则直接返回缓存的计算结果。\
> 否则重新计算、保存并返回。

#### useCallback

挂载阶段
> 创建hook。\
> 将回调以及依赖保存到hook的`memorizedState`中。\
> 返回回调。

更新阶段
> 获取hook。\
> 如果新的依赖不为空且和之前的依赖一致，则直接返回回调。\
> 否则重新保存并返回。

#### useRef

挂载阶段
> 创建hook。\
> 将{ current: ref }保存到hook的`memorizedState`。\
> 返回上述对象。

更新阶段
> 获取hook。\
> 直接返回hook的`memorizedState`。

#### useEffect

挂载阶段
> 创建hook。\
> 将effect添加到workInProgress的`updateQueue.lastEffect`上，为环状链表。该effect标记为`HookPassive | HookHasEffect`。并且也保存到hook的`memorizedState`。

更新阶段
> 获取hook。\
> 如果新的依赖不为空且未发生改变，则重新添加effect，但tag为`HookPassive`。\
> 否则重新添加effect，tag为`HookPassive | HookHasEffect`。之后会重新触发。

#### useLayoutEffect

基本和`useEffect`一致，唯一区别在effect的tag为`HookLayout`。

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

## diff

> `newChild`为单个元素。
>> 遍历`current.child`。\
>> 如果`key`不等于新元素的`key`，则将其添加到父fiber节点的`deletions`中，并将父fiber节点打上`ChildDeletion`标记。\
>> 如果`key`相等，但是`type`不相等，则将其以及后续fiber节点删除。\
>> 如果`key`相等，且`type`相等，则复用该fiber节点，其余的删除。\
>> 如果没能复用，则直接创建新的fiber节点。
>>
> `newChild`为数组。
>> 前面`index`小于第一个旧fiber的`index`的新元素，如果新元素为字符串或数字或者`key`为null，则直接创建fiber，否则中断第一次循环。\
>> 旧fiber对应字符串或数字，旧fiber和新元素的`key`不相等，则无法复用，中断第一次循环。\
>> 如果新元素都已经创建好fiber节点，则删除剩余的旧fiber节点。\
>> 如果没有更多旧fiber节点可以复用，则剩余的新元素直接创建fiber节点。\
>> 将剩余的旧fiber节点创建map，以`key`或者`index`为键。\
>> 接下来尽可能复用旧fiber节点，剩余的则删除。
