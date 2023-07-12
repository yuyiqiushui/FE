## 简单说一说你对vuex理解？
![vuex](../assets/vuex.png)
### 分析
此题考查实践能力，能说出用法只能60分。更重要的是对vuex设计理念和实现原理的解读。



### 回答策略：3w1h

1. 首先给vuex下一个定义
2. vuex解决了哪些问题，解读理念
3. 什么时候我们需要vuex
4. 你的具体用法
5. 简述原理，提升层级



首先是官网定义：

> Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用**集中式**存储管理应用的所有组件的状态，并以相应的规则保证状态以一种**可预测**的方式发生变化。



### 回答范例：

1. vuex是vue专用的状态管理库。它以全局方式集中管理应用的状态，并且可以保证状态变更的可预测性。
2. vuex主要解决的问题是多组件之间状态共享的问题，利用各种组件通信方式，我们虽然能够做到状态共享，但是往往需要在多个组件之间保持状态的一致性，这种模式很容易出现问题，也会使程序逻辑变得复杂。vuex通过把组件的共享状态抽取出来，以全局单例模式管理，这样任何组件都能用一致的方式获取和修改状态，响应式的数据也能够保证简洁的单向数据流动，我们的代码将变得更结构化且易维护。
3. vuex并非必须的，它帮我们管理共享状态，但却带来更多的概念和框架。如果我们不打算开发大型单页应用或者我们的应用并没有大量全局的状态需要维护，完全没有使用vuex的必要。一个简单的[store 模式](https://cn.vuejs.org/v2/guide/state-management.html#简单状态管理起步使用)就足够了。反之，Vuex 将会成为自然而然的选择。引用 Redux 的作者 Dan Abramov 的话说就是：Flux 架构就像眼镜：您自会知道什么时候需要它。
4. 我在使用vuex过程中有如下理解：首先是对核心概念的理解和运用，将全局状态放入state对象中，它本身一棵状态树，组件中使用store实例的state访问这些状态；然后有配套的mutation方法修改这些状态，并且只能用mutation修改状态，在组件中调用commit方法提交mutation；如果应用中有异步操作或者复杂逻辑组合，我们需要编写action，执行结束如果有状态修改仍然需要提交mutation，组件中调用这些action使用dispatch方法派发。最后是模块化，通过modules选项组织拆分出去的各个子模块，在访问状态时注意添加子模块的名称，如果子模块有设置namespace，那么在提交mutation和派发action时还需要额外的命名空间前缀。
5. vuex在实现单项数据流时需要做到数据的响应式，通过源码的学习发现是借用了vue的数据响应化特性实现的，它会利用Vue将state作为data对其进行响应化处理，从而使得这些状态发生变化时，能够导致组件重新渲染。





### 使用 Vuex 只需执行 Vue.use(Vuex),并在 Vue 的配置中传入一个 store 对象的示例， store 是如何实现诸注入的？

Vue.use(Vuex)方法执行的是 install 方法，它实现了 Vue 实例对象的 init 方法封装和注入，使传入的 store 对象被设置到 Vue 上下文环境的 &store 中。

因此 在 Vue Component 任意地方都能够通过 this.$store 访问到该 store



### state 内部支持模块配置和模块嵌套，如何实现的？

在 store 构造方法中有  makeLocalContext 方法，所有 module 都会有一个 local context，根据配置时的 path 进行匹配。所以执行如 dispatch('submitOrder', payload) 这类 action 时，默认的拿到都是 module 的 local  state， 如果要访问最外层或者是其他  module 的 state，只能从 root$state 按照 path 路径逐步进行访问。



### 在执行 dispatch 触发 action （ commit 同理 ）的时候，只需传入  ( type, payload ), action 执行函数中的第一个参数  store 是那里获取的？

 store 初始化时，所有配置的  action 和 mutation 以及 getters 均被封装过。在执行如 dispatch('submitOrder', payload) 的时候， action 中 type 为 submitOrder 的所有处理方法都是     被封装后的其第一个参数为 当前的 store 对象，所以能够 获取到 { dispatch， commit，state，rootState } 等数据。



### Vuex 如何区分 state 是外部直接修改，还是通过 mutation 方法修改的？

Vuex 中修改 state 的唯一渠道就是执行  commit('xx', payload) 方法，其底层通过执行  this._withCommit(fn) 设置  _committing  标志变量为  true， 然后才能修改  state，修改完毕还需要还原  _committing 变量。外部修改 虽然能够直接修改 state，但是并没有修改  _committing 标志位，所以只要 watch 一下 state， state change 时判断是否  _committing 值为 true，即可判断修改的合法性



### 调试的时候“时空穿梭” 功能是如何实现的？

devtoolPlugin 提供了此功能。因为 dev 模式下所有的  state  change 都会被记录下来， “时空穿梭”功能其实就是将 当前的 state 替换为  记录中某个时刻的 state 状态，利用  store.replaceState(targetState) 方法将执行 this._vm._state = state  实现
