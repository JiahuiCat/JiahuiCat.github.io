---
layout: post
title: "提高数据存储的性能，可靠性和可扩展性"
date:         2024-07-01
subtitle: "增加备用光纤存储器"
author: "Hy.Alva"
header-style: text
tags:
  - 服务器虚拟化系统存储设备
  - 光纤存储器
  - 冗余
  - 高可用性
  - 高可靠性
---

> 部分内容转自ChatGPT


**[心智模型](https://www.zhihu.com/search?q=%E5%BF%83%E6%99%BA%E6%A8%A1%E5%9E%8B&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1125724740%7D)（Mental Model）。**在网络规划中增加备用光纤存储器（例如，使用光纤通道（Fiber Channel）连接的存储区域网络（SAN））可以提高数据存储的性能、可靠性和可扩展性.

正文开始前我先声明一下，

1. 对部分网络结构细节并不太熟悉，欢迎大家务必指正、补充（与要求补充）；本文观点不代表公司。

需求分析 
--

业务需求： 

*  确定需要存储的数据类型和容量。 

*  评估存储性能需求，如读写速度和延迟要求。 

*  确定数据备份和恢复要求。 

预算和资源： 

*  确定预算，包括存储设备、光纤通道交换机和光纤线缆等。 

*  估算长期维护和扩展成本。 



设备选择和规划
-------------------------------------

光纤存储设备：

*  选择适当的光纤存储设备，如光纤通道硬盘阵列或全闪存阵列。

*  考虑设备的性能、扩展性和兼容性。

光纤通道交换机：

*  选择支持所需端口数量和速率（如8Gbps、16Gbps或32Gbps）的光纤通道交换机。

*  确保交换机支持冗余连接和高可用性。

光纤线缆和光模块：

*  选择合适的光纤线缆（单模或多模）和光模块（SFP+、QSFP等）。

*  确保线缆长度和质量满足需求。

React, "Purely (Semi-Monadic/Algebraic) FP"
-------------------------------------------

再来看 React（本文中的 React 主要指 Hooks API 下的 React）

```ts
function Counter(props) {
  const [count, setCount] = React.useState(0);
  const inc = () => setCount(count + 1)
  return <a onClick={inc}>{count}</a>
}
```

React 为组件挑选的语义是「函数」，每次渲染都是对 `Counter` 这个函数的一次真实调用。每次 `useState` 都会执行并从 React 那取出当前的状态给 `count`，每次也都会创建一个新的 `inc` 函数（故其闭包中捕获的也是新的 `count` 值）。

React 附加的核心语义是一个副作用受控的「执行上下文（evaluation context）」，通俗得说就是 React 这个运行环境：状态 `count` 每次都要从 React 上下文中取出，`inc` 对状态更改的方式是用 `setCount` 更新上下文里的内容，状态更改的结果是这个函数会被重新调用，调用时函数就会从新的上下文中获得新的状态、进行重渲染和安排上（schedule）受上下文控制的副作用（`useEffect`) 。

为了帮助你理解，如果我们有一门假想的 React 语言的话……


```ts
// hypothetical React lang Ⅰ
component Counter = (props) =>      // function
  @context.state(1) {                  // context provides `get_or` and `put`
    count <- get_or(0)              // get from context (or use default value)
    let inc = () => put(count + 1)  // callback can update the context
    return <a onClick={inc}>{count}</a>
  }
```

是不是有基于 Monad 的 Haskell 内味了？只不过 React 把 API 做得完全不需要你弄懂这些复杂的东西[\[4\]](#ref_4)。如果你不熟悉 Monad 这个纯 FP 的概念，我们可以先不严谨[\[5\]](#ref_5)得把它当做文中的「上下文」。扔出 M-bomb 的原因是大家通常把它作为在纯 FP 中处理副作用的标杆，帮助我们展示如何把 React 归约到纯 FP。

有的同学（比如[@题叶](//www.zhihu.com/people/790dccce26904cdcd11b0fad3bac37b7)）会疑惑，这怎么跟我认得的「纯函数」不一样呢，这也是「纯函数式编程」吗？其实如果我们把语法糖展开就会变成：

```ts
component Counter = (props) =>
  context.state(1).get_or(0).then([count, put] => {  // `then` is monadic bind.
    let inc = () => put(count + 1)
    return <a onClock={inc}>{count}</a>
  }).unwrap() // assuming it's safe.
```

有没有想到同样被称作 Monad，具备异步上下文的 Promise？

再（过度）简化一些，你可以想象成最直白的 state-passing 风格（实际上 2018 年时 React 团队就这么考虑过类似的 API [\[6\]](#ref_6)，也是 Seb 的理论基础之一[\[7\]](#ref_7)）:

```ts
component Counter = (props, stateMap) => {
  let count = stateMap.get(1, or=0);
  let inc = () => stateMap.set(1, count + 1); // functional update
  return <a onClick={inc}>{count}</a>
}
```


不过，React 从实现到 API 设计都更靠近与追求[\[8\]](#ref_8)的心智模型是一个相对较新的纯 FP 概念 —— Algebraic Effect（代数作用），虽然名字听起来相当迷惑，但其实它在描述副作用上比 Monad 反而更不花哨（少一些 ceremony），理解起来也更加容易，Dan 有一篇给 JSer 看的很易懂的博文[\[9\]](#ref_9)并且有中文翻译[\[10\]](#ref_10)。我们可以先把它当作「可以重新恢复的 `try-catch` 」

为了帮助你理解，如果我们又又又又有一门假想的 React 语言的话……

```ts
// hypothetical React lang Ⅱ
component Counter = (props) => {
  let count = perform getState(1),or=0); // 1. `perform` "throw" effects to the context
                                                // 4. resume with the continuation to here
  let inc = () => perform putState(1, s=count + 1);
  return <a onClick={inc}>{count}</a>
}

// call site
try <Counter /> handle  // 2.try-handle pattern match effects
                        // 3. get state from the context and then resume
| getState(key, or) => resume with context.state(key) ?? or
| putState(key, s) => context.state(key)=s; resume with void
```

是不是有感觉一些了？我们从组件里「扔出去」去更改 「执行上下文」里的状态，然后再「恢复」回来……

即便 React 已经很努力的降低了 API 的门槛，但其思维的愈加纯函数式确实会在更多程序员眼里非常「离经叛道」。

所以为什么我们需要「纯函数式编程」？抛开大家可能已经熟悉的声明式、数据流清晰、局部推理、易于组合外，其背后的学术理论支撑使得其在编译期静态分析与优化、运行时高并发高并行友好方面都有极高的理论上限和上升空间（近年编程原理的理论研究完全是被函数式编程霸占得）

React 现在运行时侧重的核心是 cooperative multitasking（协作式多任务），来为 React 加入 concurrency（并发）、schedule（调度）等底层能力。很多同学只听说过后端的高并发，其实像多任务操作系统这样的「终极 UI」就是一个高并发且依赖诸如分时（time-slicing）、优先处理（re-priortizing）等进程调度的场景。React 希望把这些技术带给 UI 开发者（比如 Suspense，比如 Selective Hydration，比如 RN 新架构中的 Fabrics），第一步是运行时用 Fiber 架构重写不依赖原生调用栈，第二步就是用 Hooks 解决 Class API 在纯度上约束力不够的问题。不纯的组件在 React 并发模式下很容易出现数据竞态（data race）的问题。

总结起来，React 在组件方面的心智模型是「副作用受上下文托管的纯函数」，只要照着这个思路去想，就比较好理解「为什么 React 中倾向于使用不可变数据结构」、「为什么 useEffect 默认会执行 cleanup 来保持幂等性」、「为什么 React 需要 `useRef` 这样的跨渲染 ref cell 机制来做 mutable ref 」、「为什么 React 的性能优化是 `useMemo`, `Memo` 这样 FP 风格的 memoization 」、「为什么 React 需要 `useMemo` `useCallback` 来保持 referential identity 」、「为什么 React 需要用依赖列表来进行 cache invalidation」等问题了。



补充
--

2016 年底时我就觉得 React 和 Vue 的（一个）终极区别在于「可变」还是「不可变」。

![](https://pica.zhimg.com/50/v2-22e50932c60262bf381456de3017b217_720w.jpg?source=1940ef5c)

Seb 在 Hooks 发布后收到一些质疑的 brain dump[\[11\]](#ref_11) 里写到：

> It's interesting because there are really two approaches evolving. There's a **mutable + change tracking** approach and there's an **immutability + referential equality testing** approach. It's difficult to mix and match them when you build new features on top. So that's why React has been pushing a bit harder on immutability lately to be able to build on top of it. Both have various tradeoffs but others are doing good research in other areas, so we've decided to focus on this direction and see where it leads us.

不全部翻译了，说得是整个「大前端」社区里最主要的两条道路分歧：

* 可变 + 变更追踪。包括 Vue，Angular，
* 不可变 + 引用相等性。包括 React，Elm，(Flutter?)

这个分歧其实与我之前行文的侧重点「为组件挑选的语义」其实是对偶的：

* 前者是对传统 Imperative Programming（包括 OOP）思路的一种增强，加入了 Reactivity。
* 后者则是传统 Functional Programming 在 UI 开发领域的发扬光大（Functional Reactive Programming?)，只不过 React 是用一种比较「超越 JS 语言」的方式去实现得。

这两条道路从底子就很不同，所以才造成了 React 和 Vue 在大家眼里的渐行渐远吧。

不过最近多看了一些 Svelte、 SwiftUI[\[12\]](#ref_12)和 Jetpack Compose[\[13\]](#ref_13)也开始有了一些殊途同归的感觉，Props 无论是跟着 View 销毁还是函数参数总是暂时性的输入，States 无论跟着组件实例还是置外总必须是持久化的，至于怎么判断更新，像 `array.push` 这种 mutable state 的场景总是不好 track 得，于是就只能各显神通了：React 想通过 reference equality 自动、Vue 3 想通过 Proxy 自动，但其实只要能把 change set 搞出来就行，Vue2/Svelte/SwiftUI/Compose 这些让用户手动给提示得不是也工作得好好得吗？只要能把变更集算出来传递给视图层，那视图层就只管更新（rerender/rebuild/recomposite）就是了。

补充 2
----

如果是在重度依赖 Flux (Vuex/Redux, whatever) 的场景，可能 Vue/React 会更像是一个只负责渲染的 dumb/passive layer，这种时候上文说的 Vue/React 的差异会显得不明显，因为大部分的状态管理（state management）都已经扔到更外层去做了。

不过，考虑需要组件内聚的场景（即组件自己有私有状态，需要 self-conatined）以及 React Hooks / Vue Composition APIs 开始接管更多（除状态之外的，比如 IO）副作用，这种差异只会变得越来越明显得。



以上。

参考
--

1.  JavaScript 模块化七日谈 - 黄玄的博客 Hux Blog [https://huangxuan.me/2015/07/09/js-module-7day/](https://huangxuan.me/2015/07/09/js-module-7day/)
2.  如何理解尤雨溪在 2019 VueConf 上所讲的 UI 类框架很少使用面向对象的特性这件事？- 黄玄的回答 [https://www.zhihu.com/question/328958700/answer/714287394](https://www.zhihu.com/question/328958700/answer/714287394)
3.  前端是否适合使用面向对象的方式编程？- 黄玄的回答 [https://www.zhihu.com/question/329005869/answer/739525268](https://www.zhihu.com/question/329005869/answer/739525268)
4.  React Hooks的引入会对之后的React项目开发产生什么影响？- 黄玄的回答 [https://www.zhihu.com/question/302916879/answer/536846510](https://www.zhihu.com/question/302916879/answer/536846510)
5.  React 上下文的组合是通过调用顺序在运行时里维护一个链表而非基于参数化多态的层叠（比如 Monad Transformer）来表达，可以看到都是线性的。
6.  State-passing Style Hooks [https://mobile.twitter.com/acdlite/status/971598256454098944](https://mobile.twitter.com/acdlite/status/971598256454098944)
7.  [https://github.com/reactjs/react-basic](https://github.com/reactjs/react-basic)
8.  可以把上下文或者 Hooks 的调用视为一次 stack unwinding + resume continuation。同样，考虑 row polymorphism 也是线性的。
9.  Algebraic Effects for the Rest of Us [https://overreacted.io/algebraic-effects-for-the-rest-of-us/](https://overreacted.io/algebraic-effects-for-the-rest-of-us/)
10.  通俗易懂的代数效应 [https://overreacted.io/zh-hans/algebraic-effects-for-the-rest-of-us/](https://overreacted.io/zh-hans/algebraic-effects-for-the-rest-of-us/)
11.  Why React [https://gist.github.com/sebmarkbage/a5ef436427437a98408672108df01919](https://gist.github.com/sebmarkbage/a5ef436427437a98408672108df01919)
12.  [https://swiftwithmajid.com/2019/06/12/understanding-property-wrappers-in-swiftui/](https://swiftwithmajid.com/2019/06/12/understanding-property-wrappers-in-swiftui/)
13.  [https://developer.android.com/jetpack/compose/state#remember](https://developer.android.com/jetpack/compose/state#remember)
