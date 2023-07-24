# pinia源码分析【2】-- createPinia

## 专栏导航

[分析pinia源码之前必须知道的 API](https://juejin.cn/post/7124279061035089927)

[pinia源码分析 [1]-源码分析环境搭建](https://juejin.cn/post/7117131804229763079)

[pinia源码分析【2】-- createPinia](https://juejin.cn/post/7119788423501578277)

[pinia 源码分析【3】-- defineStore](https://juejin.cn/post/7121661056044236831)

[pinia源码分析【4】-- pinia Methods](https://juejin.cn/post/7123504805892325406)

## 前言

参考源码`pinia V2.0.14`

源码分析仓库：[github.com/vkcyan/mini…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvkcyan%2Fmini-pinia)

上一篇文章我们主要介绍了如何搭建一个pinia源码阅读环境；本文主要介绍pinia在vue3初始化阶段相关逻辑，以及如何构建pinia对象。



## 正文

根据官方文档，我们使用`pinia`首先需要是将他注册到`vue`中

```javascript
const pinia = createPinia();
app.use(pinia);

```

`createPinia`的阶段究竟做了什么，他又是如何被注册到vue中呢？我们要从`createPinia`中寻找答案。

### 源码地址

我们通过`pinia\src\index.ts`找到

```javascript
export { createPinia } from './createPinia' // `pinia\src\createPinia.ts为源码文件`

```

### createPinia函数

在函数的最开头，我们就可以看到通过`effectScope`声明了一个`ref`，并赋值给了**state**，这里的`effectScope`是高级API，未来会单独介绍，有兴趣的同学可以看一下[官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fvuejs.org%2Fapi%2Freactivity-advanced.html%23effectscope)，我们将其**简单理解为声明了一个ref并赋值给state**。

```javascript
export function createPinia(): Pinia {
    const scope = effectScope(true);
    const state = scope.run<Ref<Record<string, StateTree>>>(() =>
       ref<Record<string, StateTree>>({})
    )!;
    // 简化理解
    // const state = ref({})
    
    // ...
}
​
```

`pinia`通过`markRaw`进行包装，将其**标记为不会转化为响应式**，最终`pinia`对象被`createPinia`函数返回，执行`vue.use(pinia)`的时候便会执行`pinia`对象中的`install`函数。



### 返回值的含义以及作用

![image-20220713153012540](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c1042346e1a4b1b8f58c9e5b2898535~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

初始化的逻辑相对比较简单，只需要了解`effectScope` `markRaw`便能完全读懂，`install`阶段组成的`pinia`对象被`setActivePinia`保存了下来，而这个对象贯穿`pinia`整个生命周期，每个字段的作用在后面的源码解读中都会有所体现。



## 关于Vue2

通过`pinia`官网，我们可以了解到`pinia`支持`vue2`，不过`vue2`环境需要在使用`createPinia`之前，预先安装插件`PiniaVuePlugin`，通过`pinia`的入口文件了解到`PiniaVuePlugin`的源码入口为`pinia\src\vue2-plugin.ts`

`PiniaVuePlugin`是`vue2`插件比较主流的实现方式，**获取Vue实例，通过mixin实现数据共享**。如果了解过`vuex`的源码，相信对以下代码会十分熟悉。

```javascript
export const PiniaVuePlugin: Plugin = function (_Vue) {
    // Equivalent of
    // app.config.globalProperties.$pinia = pinia
    // pinia在vue2中的注册逻辑与vuex核心逻辑几乎一致，
    // 注入全局mixin的beforeCreate
    _Vue.mixin({
        beforeCreate() {
            const options = this.$options;
            // 在根节点通过vue.use中注册了pinia
            if (options.pinia) {
                const pinia = options.pinia as Pinia;
                // defineProperty版provided实现
                if (!(this as any)._provided实现) {
                    const provideCache = {};
                    Object.defineProperty(this, "_provided", {
                        get: () => provideCache,
                        set: (v) => Object.assign(provideCache, v),
                    });
                }
                (this as any)._provided[piniaSymbol as any] = pinia;
​
                // 首次注册变量不存在，进行存储
                if (!this.$pinia) {
                    this.$pinia = pinia;
                }
​
                // 保存Vue实例
                pinia._a = this as any;
                if (IS_CLIENT) {
                    setActivePinia(pinia);
                }
            } else if (!this.$pinia && options.parent && options.parent.$pinia) {
                // 所有子组件/页面都继承上一层的pinia
                this.$pinia = options.parent.$pinia;
            }
        },
        destroyed() {
            delete this._pStores;
        },
    });
};

```



## 关于devTool

在`createPinia`中存在这样一段代码

```javascript
if (__DEV__ && IS_CLIENT && !__TEST__) {
    pinia.use(devtoolsPlugin);
}

```

如果是开发环境，并且是浏览器环境，并且不是测试环境，就会向pinia注册`devtoolsPlugin`，也就是将`pinia`注册到浏览器插件**Vue.js devtools**中。

![image-20220713170645056](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54eb8a2621224b6fa41354c1baac4430~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 结语

**createPinia**的源码解读就全部结束了，现在我们已经了解初始化的具体流程，以及生成的pinia对象中存在什么参数，这些参数在运行阶段都会发挥它应用的价值。

下一章我们将要解析**创建以及使用pinia**的相关源码，`defindStore`函数实现逻辑，在`defindStore`中我们将会了解到`install`阶段每个字段的实际用途，以及pinia的核心响应原理。

我正在参与掘金技术社区创作者签约计划招募活动，[点击链接报名投稿](https://juejin.cn/post/7112770927082864653)。