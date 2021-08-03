# React

## 介绍

用于快速构建用户界面的JavaScript库。组件化，跨平台。

* `v16.0`： 为了解决之前大型 React 应用一次更新遍历大量虚拟 DOM 带来个卡顿问题，React 重写了核心模块 Reconciler ，启用了 Fiber 架构；为了在让节点渲染到指定容器内，更好的实现弹窗功能，推出 createPortal API；为了捕获渲染中的异常，引入 componentDidCatch 钩子，划分了错误边界。
* `v16.2`：推出 Fragment ，解决数组元素问题。
* `v16.3`：增加 React.createRef() API，可以通过 React.createRef 取得 Ref 对象。增加 React.forwardRef() API，解决高阶组件 ref 传递问题；推出新版本 context api，迎接Provider / Consumer 时代；增加 getDerivedStateFromProps 和 getSnapshotBeforeUpdate 生命周期 。
* `v16.6`：增加 React.memo() API，用于控制子组件渲染；增加 React.lazy() API 实现代码分割；增加 contextType 让类组件更便捷的使用context；增加生命周期 getDerivedStateFromError 代替 componentDidCatch 。
* `v16.8`：全新 React-Hooks 支持，使函数组件也能做类组件的一切事情。
* `v17`： 事件绑定由 document 变成 container ，移除事件池等

## JSX

编写的JSX会被babel编译成React.createElement，用于创建ReactElement，之后再用于创建Fiber节点。

`React.createElement(type, [config], [...children])`

* type: 组件类型，如果为宿主组件，如`div`等，则为字符串；如果为自定义组件，则为组件类或者函数。
* config 组件的参数，为对象。
* children: 其余参数为子组件。

> 问：老版本的 React 中，为什么写 jsx 的文件需要默认引入 React?
> 答：因为 jsx 在被 babel 编译后，会变成上述 React.createElement 形式，所以需要引入 React，防止找不到 React 引起报错。

React.createElement执行流程：

* 从`config`中提取`key`、`ref`。
* 剩余的非保留参数存放在`props`中，设置`props.children`，可能为单个元素或数组。
* 处理`defaultProps`，`props`值为undefined才进行设置。
* 创建`ReactElement`对象，包含`$$typeof`、`type`。

Fiber tag：

```js
export const FunctionComponent = 0;       // 函数组件
export const ClassComponent = 1;          // 类组件
export const IndeterminateComponent = 2;  // 初始化的时候不知道是函数组件还是类组件
export const HostRoot = 3;                // Root Fiber 可以理解为根元素， 通过reactDom.render()产生的根元素
export const HostPortal = 4;              // 对应  ReactDOM.createPortal 产生的 Portal
export const HostComponent = 5;           // dom 元素 比如 <div>
export const HostText = 6;                // 文本节点
export const Fragment = 7;                // 对应 <React.Fragment>
export const Mode = 8;                    // 对应 <React.StrictMode>
export const ContextConsumer = 9;         // 对应 <Context.Consumer>
export const ContextProvider = 10;        // 对应 <Context.Provider>
export const ForwardRef = 11;             // 对应 React.ForwardRef
export const Profiler = 12;               // 对应 <Profiler/ >
export const SuspenseComponent = 13;      // 对应 <Suspense>
export const MemoComponent = 14;          // 对应 React.memo 返回的组件
```

Fiber 树节点之间的关系：

* child: 指向子节点
* sibiling: 指向兄弟节点
* return: 指向父节点

常见操作ReactElement的API：

* React.Children.toArray: 扁平化、规范化children数组。
* React.Children.forEach: 遍历，相当于React.Children.toArray + Array.prototype.forEach。
* React.isValidElement: 是否有效。
* React.cloneElement: 克隆。
