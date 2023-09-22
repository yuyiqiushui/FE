# 微前端框架 之 qiankun 从入门到源码分析

当学习成为了习惯，知识也就变成了常识。感谢各位的 **Star**、**Watch** 和 **评论**。

新视频和文章会第一时间在微信公众号发送，欢迎关注：[李永宁lyn](https://gitee.com/liyongning/typora-image-bed/raw/master/202202171742614.jpg)

## 简介

从 single-spa 的缺陷讲起 -> qiankun 是如何从框架层面解决 single-spa 存在的问题 -> qiankun 源码解读，带你全方位刨析 qiankun 框架。

## 介绍

qiankun 是基于 single-spa 做了二次封装的微前端框架，通过解决了 single-spa 的一些弊端和不足，来帮助大家实现更简单、无痛的构建一个生产可用的微前端架构系统。

[微前端框架 之 single-spa 从入门到精通](https://mp.weixin.qq.com/s?__biz=MzA3NTk4NjQ1OQ==&mid=2247484245&idx=1&sn=9ee91018578e6189f3b11a4d688228c5&chksm=9f696021a81ee937847c962e3135017fff9ba8fd0b61f782d7245df98582a1410aa000dc5fdc&token=1002082546&lang=zh_CN#rd) 通过从 基本使用 -> 部署 -> 框架源码分析 -> 手写框架，带你全方位刨析 single-spa 框架。

因为 qiankun 是基于 single-spa 做的二次封装，主要解决了 single-spa 的一些痛点和不足，所以最好对 single-spa 有一个全面的了解和认识，明白其原理、了解它的不足和缺陷，然后带着问题和目的去阅读 qiankun 源码，可以达到事半功倍的效果，整个阅读过程的思路也会更加清晰明了。

## 为什么不是 single-spa

如果你很了解 single-spa 或者阅读过 [微前端框架 之 single-spa 从入门到精通](https://mp.weixin.qq.com/s?__biz=MzA3NTk4NjQ1OQ==&mid=2247484245&idx=1&sn=9ee91018578e6189f3b11a4d688228c5&chksm=9f696021a81ee937847c962e3135017fff9ba8fd0b61f782d7245df98582a1410aa000dc5fdc&token=1002082546&lang=zh_CN#rd) ，你会发现 single-spa 就做了两件事，加载微应用（加载方法还是用户自己提供的）、维护微应用状态（初始化、挂载、卸载）。了解多了会发现 single-spa 虽好，但是却存在一些比较严重的问题

1. **对微应用的侵入性太强**

   single-spa 采用 JS Entry 的方式接入微应用。微应用改造一般分为三步：

   - 微应用路由改造，添加一个特定的前缀
   - 微应用入口改造，挂载点变更和生命周期函数导出
   - 打包工具配置更改

   侵入型强其实说的就是第三点，更改打包工具的配置，使用 single-spa 接入微应用需要将微应用整个打包成一个 JS 文件，发布到静态资源服务器，然后在主应用中配置该 JS 文件的地址告诉 single-spa 去这个地址加载微应用。

   不说其它的，就现在这个改动就存在很大的问题，将整个微应用打包成一个 JS 文件，常见的打包优化基本上都没了，比如：按需加载、首屏资源加载优化、css 独立打包等优化措施。

   项目发布以后出现了 bug ，修复之后需要更新上线，为了清除浏览器缓存带来的影响，一般文件名会带上 chunkcontent，微应用发布之后文件名都会发生变化，这时候还需要更新主应用中微应用配置，然后重新编译主应用然后发布，这套操作简直是不能忍受的，这也是 [微前端框架 之 single-spa 从入门到精通](https://mp.weixin.qq.com/s?__biz=MzA3NTk4NjQ1OQ==&mid=2247484245&idx=1&sn=9ee91018578e6189f3b11a4d688228c5&chksm=9f696021a81ee937847c962e3135017fff9ba8fd0b61f782d7245df98582a1410aa000dc5fdc&token=1002082546&lang=zh_CN#rd) 这篇文章中示例项目中微应用发布时的环境配置选择 development 的原因。

2. **样式隔离问题**

   single-spa 没有做这部分的工作。一个大型的系统会有很的微应用组成，怎么保证这些微应用之间的样式互不影响？微应用和主应用之间的样式互不影响？这时只能通过约定命名规范来实现，比如应用样式以自己的应用名称开头，以应用名构造一个独立的命名空间，这个方式新系统还好说，如果是一个已有的系统，这个改造工作量可不小。

3. **JS 隔离**

   这部分工作 single-spa 也没有做。 JS 全局对象污染是一个很常见的现象，比如：微应用 A 在全局对象上添加了一个自己特有的属性，`window.A`，这时候切换到微应用 B，这时候如何保证 `window` 对象是干净的呢？

4. **资源预加载**

   这部分的工作 single-spa 更没做了，毕竟将微应用整个打包成一个 js 文件。现在有个需求，比如为了提高系统的用户体验，在第一个微应用挂载完成后，需要让浏览器在后台悄悄的加载其它微应用的静态资源，这个怎么实现呢？

5. **应用间通信**

   这部分工作 single-spa 没做，它只在注册微应用时给微应用注入一些状态信息，后续就不管了，没有任何通信的手段，只能用户自己去实现

以上 5 个问题中第 2、3、5 还好说，可以通过一些方式来解决，比如采用命名空间的方式解决样式隔离问题， 通过备份全局对象，每次微应用切换时初始化全局对象的方式来解决 JS 隔离的问题，通信问题可以通过传递一些通信方法，这点依赖了 JS 对象本身的特性（传递的是引用）来实现；但是第一个和第四个就不好解决了，这是 JS Entry 方式带来的问题，要解决这个问题，难度相对就会大很多，工作量也会更大。况且这些通用的脏活累活就不应该由用户（框架使用者）来解决，而是由框架来解决。

## 为什么是 qiankun

上面说到，通用的脏活累活应该在框架层面去做，qiankun 基于 single-spa 做了二次封装，很好的解决了上面提到的几个问题。

1. **HTML Entry**

   qiankun 通过 HTML Entry 的方式来解决 JS Entry 带来的问题，让你接入微应用像使用 iframe 一样简单。

2. **样式隔离**

   qiankun 实现了两种样式隔离

   - 严格的样式隔离模式，为每个微应用的容器包裹上一个 `shadow dom` 节点，从而确保微应用的样式不会对全局造成影响
   - 实验性的方式，通过动态改写 `css` 选择器来实现，可以理解为 `css scoped` 的方式

3. **运行时沙箱**

   qiankun 的运行时沙箱分为 `JS` 沙箱和 `样式沙箱`

   `JS 沙箱` 为每个微应用生成单独的 `window proxy` 对象，配合 `HTML Entry` 提供的 JS 脚本执行器 (execScripts) 来实现 JS 隔离；

   `样式沙箱` 通过重写 `DOM` 操作方法，来劫持动态样式和 JS 脚本的添加，让样式和脚本添加到正确的地方，即主应用的插入到主应用模版内，微应用的插入到微应用模版，并且为劫持的动态样式做了 `scoped css` 的处理，为劫持的脚本做了 JS 隔离的处理，更加具体的内容可继续往下阅读或者直接阅读 [微前端专栏](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA3NTk4NjQ1OQ==&action=getalbum&album_id=2251416802327232513&scene=173&from_msgid=2247484245&from_itemidx=1&count=3&nolastread=1#wechat_redirect) 中的 `qiankun 2.x 运行时沙箱 源码分析`

4. **资源预加载**

   qiankun 实现预加载的思路有两种，一种是当主应用执行 `start` 方法启动 qiankun 以后立即去预加载微应用的静态资源，另一种是在第一个微应用挂载以后预加载其它微应用的静态资源，这个是利用 single-spa 提供的 `single-spa:first-mount` 事件来实现的

5. **应用间通信**

   qiankun 通过发布订阅模式来实现应用间通信，状态由框架来统一维护，每个应用在初始化时由框架生成一套通信方法，应用通过这些方法来更改全局状态和注册回调函数，全局状态发生改变时触发各个应用注册的回调函数执行，将新旧状态传递到所有应用

## 说明

文章基于 `qiankun 2.0.26` 版本做了完整的源码分析，目前网上好像还没有 `qiankun 2.x` 版本的完整源码分析，简单搜了下好像都是 1.x 版本的

由于框架代码比较多的，博客有字数限制，所以将全部内容拆成了三篇文章，每一篇都可独立阅读：

- 微前端框架 之 qiankun 从入门到精通

  ，文章由以下三部分组成

  - `为什么不是 single-spa`，详细介绍了 single-spa 存在的问题
  - `为什么是 qiankun`，详细介绍了 qiankun 是怎么从框架层面解决 single-spa 存在的问题的
  - `源码解读`，完整解读了 qiankun 2.x 版本的源码

- [qiankun 2.x 运行时沙箱 源码分析](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA3NTk4NjQ1OQ==&action=getalbum&album_id=2251416802327232513&scene=173&from_msgid=2247484245&from_itemidx=1&count=3&nolastread=1#wechat_redirect)，详细解读了 qiankun 2.x 版本的沙箱实现

- [HTML Entry 源码分析](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA3NTk4NjQ1OQ==&action=getalbum&album_id=2251416802327232513&scene=173&from_msgid=2247484245&from_itemidx=1&count=3&nolastread=1#wechat_redirect)，详细解读了 HTML Entry 的原理以及在 qiankun 中的应用

## 源码解读

这里没有单独编写示例代码，因为 `qiankun` 源码中提供了完整的示例项目，这也是 `qiankun` 做的很好的一个地方，提供完整的示例，避免大家在使用时重复踩坑。

微前端实现和改造时面临的第一个困难就是主应用的设置、微应用的接入，`single-spa` 官方没有提供一个很好的示例项目，所以大家在使用 `single-spa` 接入微应用时还是需要踩不少坑的，甚至有些问题需要去阅读源码才能解决

### 框架目录结构

从 [github](https://github.com/liyongning/qiankun) 克隆项目以后，执行一下命令：

- 安装 `qiankun` 框架所需的包

  ```js
  yarn install
  ```

- 安装示例项目的包

  ```js
  yarn examples:install
  ```

以上命令执行结束以后：

#### 有料的 package.json

- [npm-run-all](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fnpm-run-all)

  > 一个 CLI 工具，用于并行或顺序执行多个 npm 脚本

- [father-build](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fumijs%2Ffather)

  > 基于 rollup 的库构建工具，father 更加强大

- 多项目的目录组织以及 scripts 部分的编写

- main 和 module 字段

  标识组件库的入口，当两者同时存在时，module 字段的优先级高于 main

#### 示例项目中的主应用

这里需要更改一下示例项目中主应用的 `webpack` 配置

```js
{
  ...
  devServer: {
    // 从 package.json 中可以看出，启动示例项目时，主应用执行了两条命令，其实就是启动了两个主应用，但是却只配置了一个端口，浏览器打开 localhost:7099 和你预想的有一些出入，这时显示的是 loadMicroApp(手动加载微应用) 方式的主应用，基于路由配置的主应用没起来，因为端口被占用了
    // port: '7099'
		// 这样配置，手动加载微应用的主应用在 7099 端口，基于路由配置的主应用在 7088 端口
    port: process.env.MODE === 'multiple' ? '7099' : '7088'
  }
  ...
}
```

#### 启动示例项目

```
yarn examples:start
```



命令执行结束以后，访问 `localhost:7099` 和 `localhost:7088` 两个地址，可以看到如下内容：

[![image-20220202220258551](https://camo.githubusercontent.com/4df6fc36276c7050dda2bf219f371740b71d769ba0f724db5c87d7731e70db4e/68747470733a2f2f67697465652e636f6d2f6c69796f6e676e696e672f7479706f72612d696d6167652d6265642f7261772f6d61737465722f3230323230323032323230323736302e706e67)](https://camo.githubusercontent.com/4df6fc36276c7050dda2bf219f371740b71d769ba0f724db5c87d7731e70db4e/68747470733a2f2f67697465652e636f6d2f6c69796f6e676e696e672f7479706f72612d696d6167652d6265642f7261772f6d61737465722f3230323230323032323230323736302e706e67)

[![image-20220202220401608](https://camo.githubusercontent.com/a505ad0c3835788372e5c536247df211d61025e5293acb8c95095de400ad6d2a/68747470733a2f2f67697465652e636f6d2f6c69796f6e676e696e672f7479706f72612d696d6167652d6265642f7261772f6d61737465722f3230323230323032323230343831382e706e67)](https://camo.githubusercontent.com/a505ad0c3835788372e5c536247df211d61025e5293acb8c95095de400ad6d2a/68747470733a2f2f67697465652e636f6d2f6c69796f6e676e696e672f7479706f72612d696d6167652d6265642f7261772f6d61737465722f3230323230323032323230343831382e706e67)

到这一步，就证明项目正式跑起来了，所有准备工作就绪

### 示例项目

官方为我们准备了两种主应用的实现方式，五种微应用的接入示例，覆盖面可以说是比较广了，足以满足大家的普遍需要了

#### 主应用

主应用在 `examples/main` 目录下，提供了两种实现方式，基于路由配置的 `registerMicroApps` 和 手动加载微应用的 `loadMicroApp`。主应用很简单，就是一个从 0 通过 webpack 配置的一个同时支持 react 和 vue 的项目，至于为什么同时支持 react 和 vue，继续往下看

##### webpack.config.js

就是一个普通的 `webpack` 配置，配置了一个开发服务器 `devServer`、两个 `loader` (babel-loader、css loader)、一个插件 `HtmlWebpackPlugin` (告诉 webpack html 模版文件是哪个)

通过 `webpack` 配置文件的 `entry` 字段得知入口文件分别为 `index.js` 和 `multiple.js`

##### 基于路由配置

通用将微应用关联到一些 `url` 规则的方式，实现当浏览器 `url` 发生变化时，自动加载相应的微应用的功能

###### index.js

```js
// qiankun api 引入
import { registerMicroApps, runAfterFirstMounted, setDefaultMountApp, start, initGlobalState } from '../../es';
// 全局样式
import './index.less';

// 专门针对 angular 微应用引入的一个库
import 'zone.js';

/**
 * 主应用可以使用任何技术栈，这里提供了 react 和 vue 两种，可以随意切换
 * 最终都导出了一个 render 函数，负责渲染主应用
 */
// import render from './render/ReactRender';
import render from './render/VueRender';

// 初始化主应用，其实就是渲染主应用
render({ loading: true });

// 定义 loader 函数，切换微应用时由 qiankun 框架负责调用显示一个 loading 状态
const loader = loading => render({ loading });

// 注册微应用
registerMicroApps(
  // 微应用配置列表
  [
    {
      // 应用名称
      name: 'react16',
      // 应用的入口地址
      entry: '//localhost:7100',
      // 应用的挂载点，这个挂载点在上面渲染函数中的模版里面提供的
      container: '#subapp-viewport',
      // 微应用切换时调用的方法，显示一个 loading 状态
      loader,
      // 当路由前缀为 /react16 时激活当前应用
      activeRule: '/react16',
    },
    {
      name: 'react15',
      entry: '//localhost:7102',
      container: '#subapp-viewport',
      loader,
      activeRule: '/react15',
    },
    {
      name: 'vue',
      entry: '//localhost:7101',
      container: '#subapp-viewport',
      loader,
      activeRule: '/vue',
    },
    {
      name: 'angular9',
      entry: '//localhost:7103',
      container: '#subapp-viewport',
      loader,
      activeRule: '/angular9',
    },
    {
      name: 'purehtml',
      entry: '//localhost:7104',
      container: '#subapp-viewport',
      loader,
      activeRule: '/purehtml',
    },
  ],
  // 全局生命周期钩子，切换微应用时框架负责调用
  {
    beforeLoad: [
      app => {
        // 这个打印日志的方法可以学习一下，第三个参数会替换掉第一个参数中的 %c%s，并且第三个参数的颜色由第二个参数决定
        console.log('[LifeCycle] before load %c%s', 'color: green;', app.name);
      },
    ],
    beforeMount: [
      app => {
        console.log('[LifeCycle] before mount %c%s', 'color: green;', app.name);
      },
    ],
    afterUnmount: [
      app => {
        console.log('[LifeCycle] after unmount %c%s', 'color: green;', app.name);
      },
    ],
  },
);

// 定义全局状态，并返回两个通信方法
const { onGlobalStateChange, setGlobalState } = initGlobalState({
  user: 'qiankun',
});

// 监听全局状态的更改，当状态发生改变时执行回调函数
onGlobalStateChange((value, prev) => console.log('[onGlobalStateChange - master]:', value, prev));

// 设置新的全局状态，只能设置一级属性，微应用只能修改已存在的一级属性
setGlobalState({
  ignore: 'master',
  user: {
    name: 'master',
  },
});

// 设置默认进入的子应用，当主应用启动以后默认进入指定微应用
setDefaultMountApp('/react16');

// 启动应用
start();

// 当第一个微应用挂载以后，执行回调函数，在这里可以做一些特殊的事情，比如开启一监控或者买点脚本
runAfterFirstMounted(() => {
  console.log('[MainApp] first app mounted');
});
```

###### VueRender.js

```js
/**
 * 导出一个由 vue 实现的渲染函数，渲染了一个模版，模版里面包含一个 loading 状态节点和微应用容器节点
 */
import Vue from 'vue/dist/vue.esm';

// 返回一个 vue 实例
function vueRender({ loading }) {
  return new Vue({
    template: `
      <div id="subapp-container">
        <h4 v-if="loading" class="subapp-loading">Loading...</h4>
        <div id="subapp-viewport"></div>
      </div>
    `,
    el: '#subapp-container',
    data() {
      return {
        loading,
      };
    },
  });
}

// vue 实例
let app = null;

// 渲染函数
export default function render({ loading }) {
  // 单例，如果 vue 实例不存在则实例化主应用，存在则说明主应用已经渲染，需要更新主营应用的 loading 状态
  if (!app) {
    app = vueRender({ loading });
  } else {
    app.loading = loading;
  }
}
```



###### ReactRender.js

```js
/**
 * 同 vue 实现的渲染函数，这里通过 react 实现了一个一样的渲染函数
 */
import React from 'react';
import ReactDOM from 'react-dom';

// 渲染主应用
function Render(props) {
  const { loading } = props;

  return (
    <>
      {loading && <h4 className="subapp-loading">Loading...</h4>}
      <div id="subapp-viewport" />
    </>
  );
}

// 将主应用渲染到指定节点下
export default function render({ loading }) {
  const container = document.getElementById('subapp-container');
  ReactDOM.render(<Render loading={loading} />, container);
}
```



##### 手动加载微应用

通常这种场景下的微应用是一个不带路由的可独立运行的业务组件，这种使用方式的情况比较少见

###### multiple.js

```js
/**
 * 调用 loadMicroApp 方法注册了两个微应用
 */
import { loadMicroApp } from '../../es';

const app1 = loadMicroApp(
  // 应用配置，名称、入口地址、容器节点
  { name: 'react15', entry: '//localhost:7102', container: '#react15' },
  // 可以添加一些其它的配置，比如：沙箱、样式隔离等
  {
    sandbox: {
      // strictStyleIsolation: true,
    },
  },
);

const app2 = loadMicroApp(
  { name: 'vue', entry: '//localhost:7101', container: '#vue' },
  {
    sandbox: {
      // strictStyleIsolation: true,
    },
  },
);
```



#### vue

vue 微应用在 `examples/vue` 目录下，就是一个通过 vue-cli 创建的 vue demo 应用，然后对 `vue.config.js` 和 `main.js` 做了一些更改

##### vue.config.js

一个普通的 `webpack` 配置，需要注意的地方就三点

```js
{
  ...
  // publicPath 没在这里设置，是通过 webpack 提供的全局变量 __webpack_public_path__ 来即时设置的，webpackjs.com/guides/public-path/
  devServer: {
    ...
    // 设置跨域，因为主应用需要通过 fetch 去获取微应用引入的静态资源的，所以必须要求这些静态资源支持跨域
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  },
  output: {
    // 把子应用打包成 umd 库格式
    library: `${name}-[name]`,	// 库名称，唯一
    libraryTarget: 'umd',
    jsonpFunction: `webpackJsonp_${name}`,
  }
  ...
}
```



##### main.js

```js
// 动态设置 __webpack_public_path__
import './public-path';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import Vue from 'vue';
import VueRouter from 'vue-router';
import App from './App.vue';
// 路由配置
import routes from './router';
import store from './store';

Vue.config.productionTip = false;

Vue.use(ElementUI);

let router = null;
let instance = null;

// 应用渲染函数
function render(props = {}) {
  const { container } = props;
  // 实例化 router，根据应用运行环境设置路由前缀
  router = new VueRouter({
    // 作为微应用运行，则设置 /vue 为前缀，否则设置 /
    base: window.__POWERED_BY_QIANKUN__ ? '/vue' : '/',
    mode: 'history',
    routes,
  });

  // 实例化 vue 实例
  instance = new Vue({
    router,
    store,
    render: h => h(App),
  }).$mount(container ? container.querySelector('#app') : '#app');
}

// 支持应用独立运行
if (!window.__POWERED_BY_QIANKUN__) {
  render();
}

/**
 * 从 props 中获取通信方法，监听全局状态的更改和设置全局状态，只能操作一级属性
 * @param {*} props 
 */
function storeTest(props) {
  props.onGlobalStateChange &&
    props.onGlobalStateChange(
      (value, prev) => console.log(`[onGlobalStateChange - ${props.name}]:`, value, prev),
      true,
    );
  props.setGlobalState &&
    props.setGlobalState({
      ignore: props.name,
      user: {
        name: props.name,
      },
    });
}

/**
 * 导出的三个生命周期函数
 */
// 初始化
export async function bootstrap() {
  console.log('[vue] vue app bootstraped');
}

// 挂载微应用
export async function mount(props) {
  console.log('[vue] props from main framework', props);
  storeTest(props);
  render(props);
}

// 卸载、销毁微应用
export async function unmount() {
  instance.$destroy();
  instance.$el.innerHTML = '';
  instance = null;
  router = null;
}
```



##### public-path.js

```js
/**
 * 在入口文件中使用 ES6 模块导入，则在导入后对 __webpack_public_path__ 进行赋值。
 * 在这种情况下，必须将公共路径(public path)赋值移至专属模块，然后将其在最前面导入
 */

// qiankun 设置的全局变量，表示应用作为微应用在运行
if (window.__POWERED_BY_QIANKUN__) {
  // eslint-disable-next-line no-undef
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```



#### jQuery

这是一个使用了 jQuery 的项目，在 `examples/purehtml` 目录下，展示了如何接入使用 jQuery 开发的应用

##### package.json

为了达到演示效果，使用 `http-server` 在起了一个本地服务器，并且支持跨域

```js
{
  ...
  "scripts": {
    "start": "cross-env PORT=7104 http-server . --cors",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  ...
}
```



##### entry.js

```js
// 渲染函数
const render = $ => {
  $('#purehtml-container').html('Hello, render with jQuery');
  return Promise.resolve();
};

// 在全局对象上导出三个生命周期函数
(global => {
  global['purehtml'] = {
    bootstrap: () => {
      console.log('purehtml bootstrap');
      return Promise.resolve();
    },
    mount: () => {
      console.log('purehtml mount');
      // 调用渲染函数
      return render($);
    },
    unmount: () => {
      console.log('purehtml unmount');
      return Promise.resolve();
    },
  };
})(window);
```



##### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Purehtml Example</title>
  <script src="//cdn.bootcss.com/jquery/3.4.1/jquery.min.js">
  </script>
</head>
<body>
  <div style="display: flex; justify-content: center; align-items: center; height: 200px;">
    Purehtml Example
  </div>
  <div id="purehtml-container" style="text-align:center"></div>
  <!-- 引入 entry.js，相当于 vue 项目的 publicPath 配置 -->
  <script src="//localhost:7104/entry.js" entry></script>
</body>
</html>
```



#### angular 9、react 15、react 16

这三个实例项目就不一一分析了，和 vue 项目类似，都是配置打包工具将微应用打包成一个 umd 格式，然后配置应用入口文件 和 路由前缀

#### 小结

好了，读到这里，系统改造（可以开始干活了）基本上就已经可以顺利进行了，从主应用的开发到微应用接入，应该是不会有什么问题了。

当然如果你想继续深入了解，比如：

- 上面用到那些 API 的原理是什么？
- qiankun 是怎么解决我们之前提到的 single-spa 未解决的问题的？
- ...

接下来就带着我们的疑问和目的去全面深入的了解 `qiankun` 框架的内部实现

### 框架源码

整个框架的源码目录是 `src`，入口文件是 `src/index.ts`

#### 入口 src/index.ts

```
/**
 * 在示例或者官网提到的所有 API 都在这里统一导出
 */
// 最关键的三个，手动加载微应用、基于路由配置、启动 qiankun
export { loadMicroApp, registerMicroApps, start } from './apis';
// 全局状态
export { initGlobalState } from './globalState';
// 全局的未捕获异常处理器
export * from './errorHandler';
// setDefaultMountApp 设置主应用启动后默认进入哪个微应用、runAfterFirstMounted 设置当第一个微应用挂载以后需要调用的一些方法
export * from './effects';
// 类型定义
export * from './interfaces';
// prefetch
export { prefetchImmediately as prefetchApps } from './prefetch';
```

#### registerMicroApps

```
/**
 * 注册微应用，基于路由配置
 * @param apps = [
 *  {
 *    name: 'react16',
 *    entry: '//localhost:7100',
 *    container: '#subapp-viewport',
 *    loader,
 *    activeRule: '/react16'
 *  },
 *  ...
 * ]
 * @param lifeCycles = { ...各个生命周期方法对象 }
 */
export function registerMicroApps<T extends object = {}>(
  apps: Array<RegistrableApp<T>>,
  lifeCycles?: FrameworkLifeCycles<T>,
) {
  // 防止微应用重复注册，得到所有没有被注册的微应用列表
  const unregisteredApps = apps.filter(app => !microApps.some(registeredApp => registeredApp.name === app.name));

  // 所有的微应用 = 已注册 + 未注册的(将要被注册的)
  microApps = [...microApps, ...unregisteredApps];

  // 注册每一个微应用
  unregisteredApps.forEach(app => {
    // 注册时提供的微应用基本信息
    const { name, activeRule, loader = noop, props, ...appConfig } = app;

    // 调用 single-spa 的 registerApplication 方法注册微应用
    registerApplication({
      // 微应用名称
      name,
      // 微应用的加载方法，Promise<生命周期方法组成的对象>
      app: async () => {
        // 加载微应用时主应用显示 loading 状态
        loader(true);
        // 这句可以忽略，目的是在 single-spa 执行这个加载方法时让出线程，让其它微应用的加载方法都开始执行
        await frameworkStartedDefer.promise;

        // 核心、精髓、难点所在，负责加载微应用，然后一大堆处理，返回 bootstrap、mount、unmount、update 这个几个生命周期
        const { mount, ...otherMicroAppConfigs } = await loadApp(
          // 微应用的配置信息
          { name, props, ...appConfig },
          // start 方法执行时设置的配置对象
          frameworkConfiguration,
          // 注册微应用时提供的全局生命周期对象
          lifeCycles,
        );

        return {
          mount: [async () => loader(true), ...toArray(mount), async () => loader(false)],
          ...otherMicroAppConfigs,
        };
      },
      // 微应用的激活条件
      activeWhen: activeRule,
      // 传递给微应用的 props
      customProps: props,
    });
  });
}
```



#### start

```
/**
 * 启动 qiankun
 * @param opts start 方法的配置对象 
 */
export function start(opts: FrameworkConfiguration = {}) {
  // qiankun 框架默认开启预加载、单例模式、样式沙箱
  frameworkConfiguration = { prefetch: true, singular: true, sandbox: true, ...opts };
  // 从这里可以看出 start 方法支持的参数不止官网文档说的那些，比如 urlRerouteOnly，这个是 single-spa 的 start 方法支持的
  const { prefetch, sandbox, singular, urlRerouteOnly, ...importEntryOpts } = frameworkConfiguration;

  // 预加载
  if (prefetch) {
    // 执行预加载策略，参数分别为微应用列表、预加载策略、{ fetch、getPublicPath、getTemplate }
    doPrefetchStrategy(microApps, prefetch, importEntryOpts);
  }

  // 样式沙箱
  if (sandbox) {
    if (!window.Proxy) {
      console.warn('[qiankun] Miss window.Proxy, proxySandbox will degenerate into snapshotSandbox');
      // 快照沙箱不支持非 singular 模式
      if (!singular) {
        console.error('[qiankun] singular is forced to be true when sandbox enable but proxySandbox unavailable');
        // 如果开启沙箱，会强制使用单例模式
        frameworkConfiguration.singular = true;
      }
    }
  }

  // 执行 single-spa 的 start 方法，启动 single-spa
  startSingleSpa({ urlRerouteOnly });

  frameworkStartedDefer.resolve();
}
```



#### 预加载 - doPrefetchStrategy

```
/**
 * 执行预加载策略，qiankun 支持四种
 * @param apps 所有的微应用 
 * @param prefetchStrategy 预加载策略，四种 =》 
 *  1、true，第一个微应用挂载以后加载其它微应用的静态资源，利用的是 single-spa 提供的 single-spa:first-mount 事件来实现的
 *  2、string[]，微应用名称数组，在第一个微应用挂载以后加载指定的微应用的静态资源
 *  3、all，主应用执行 start 以后就直接开始预加载所有微应用的静态资源
 *  4、自定义函数，返回两个微应用组成的数组，一个是关键微应用组成的数组，需要马上就执行预加载的微应用，一个是普通的微应用组成的数组，在第一个微应用挂载以后预加载这些微应用的静态资源
 * @param importEntryOpts = { fetch, getPublicPath, getTemplate }
 */
export function doPrefetchStrategy(
  apps: AppMetadata[],
  prefetchStrategy: PrefetchStrategy,
  importEntryOpts?: ImportEntryOpts,
) {
  // 定义函数，函数接收一个微应用名称组成的数组，然后从微应用列表中返回这些名称所对应的微应用，最后得到一个数组[{name, entry}, ...]
  const appsName2Apps = (names: string[]): AppMetadata[] => apps.filter(app => names.includes(app.name));

  if (Array.isArray(prefetchStrategy)) {
    // 说明加载策略是一个数组，当第一个微应用挂载之后开始加载数组内由用户指定的微应用资源，数组内的每一项表示一个微应用的名称
    prefetchAfterFirstMounted(appsName2Apps(prefetchStrategy as string[]), importEntryOpts);
  } else if (isFunction(prefetchStrategy)) {
    // 加载策略是一个自定义的函数，可完全自定义应用资源的加载时机（首屏应用、次屏应用)
    (async () => {
      // critical rendering apps would be prefetch as earlier as possible，关键的应用程序应该尽可能早的预取
      // 执行加载策略函数，函数会返回两个数组，一个关键的应用程序数组，会立即执行预加载动作，另一个是在第一个微应用挂载以后执行微应用静态资源的预加载
      const { criticalAppNames = [], minorAppsName = [] } = await prefetchStrategy(apps);
      // 立即预加载这些关键微应用程序的静态资源
      prefetchImmediately(appsName2Apps(criticalAppNames), importEntryOpts);
      // 当第一个微应用挂载以后预加载这些微应用的静态资源
      prefetchAfterFirstMounted(appsName2Apps(minorAppsName), importEntryOpts);
    })();
  } else {
    // 加载策略是默认的 true 或者 all
    switch (prefetchStrategy) {
      case true:
        // 第一个微应用挂载之后开始加载其它微应用的静态资源
        prefetchAfterFirstMounted(apps, importEntryOpts);
        break;

      case 'all':
        // 在主应用执行 start 以后就开始加载所有微应用的静态资源
        prefetchImmediately(apps, importEntryOpts);
        break;

      default:
        break;
    }
  }
}

// 判断是否为弱网环境
const isSlowNetwork = navigator.connection
  ? navigator.connection.saveData ||
    (navigator.connection.type !== 'wifi' &&
      navigator.connection.type !== 'ethernet' &&
      /(2|3)g/.test(navigator.connection.effectiveType))
  : false;

/**
 * prefetch assets, do nothing while in mobile network
 * 预加载静态资源，在移动网络下什么都不做
 * @param entry
 * @param opts
 */
function prefetch(entry: Entry, opts?: ImportEntryOpts): void {
  // 弱网环境下不执行预加载
  if (!navigator.onLine || isSlowNetwork) {
    // Don't prefetch if in a slow network or offline
    return;
  }

  // 通过时间切片的方式去加载静态资源，在浏览器空闲时去执行回调函数，避免浏览器卡顿
  requestIdleCallback(async () => {
    // 得到加载静态资源的函数
    const { getExternalScripts, getExternalStyleSheets } = await importEntry(entry, opts);
    // 样式
    requestIdleCallback(getExternalStyleSheets);
    // js 脚本
    requestIdleCallback(getExternalScripts);
  });
}

/**
 * 在第一个微应用挂载之后开始加载 apps 中指定的微应用的静态资源
 * 通过监听 single-spa 提供的 single-spa:first-mount 事件来实现，该事件在第一个微应用挂载以后会被触发
 * @param apps 需要被预加载静态资源的微应用列表，[{ name, entry }, ...]
 * @param opts = { fetch , getPublicPath, getTemplate }
 */
function prefetchAfterFirstMounted(apps: AppMetadata[], opts?: ImportEntryOpts): void {
  // 监听 single-spa:first-mount 事件
  window.addEventListener('single-spa:first-mount', function listener() {
    // 已挂载的微应用
    const mountedApps = getMountedApps();
    // 从预加载的微应用列表中过滤出未挂载的微应用
    const notMountedApps = apps.filter(app => mountedApps.indexOf(app.name) === -1);

    // 开发环境打印日志，已挂载的微应用和未挂载的微应用分别有哪些
    if (process.env.NODE_ENV === 'development') {
      console.log(`[qiankun] prefetch starting after ${mountedApps} mounted...`, notMountedApps);
    }

    // 循环加载微应用的静态资源
    notMountedApps.forEach(({ entry }) => prefetch(entry, opts));

    // 移除 single-spa:first-mount 事件
    window.removeEventListener('single-spa:first-mount', listener);
  });
}

/**
 * 在执行 start 启动 qiankun 之后立即预加载所有微应用的静态资源
 * @param apps 需要被预加载静态资源的微应用列表，[{ name, entry }, ...]
 * @param opts = { fetch , getPublicPath, getTemplate }
 */
export function prefetchImmediately(apps: AppMetadata[], opts?: ImportEntryOpts): void {
  // 开发环境打印日志
  if (process.env.NODE_ENV === 'development') {
    console.log('[qiankun] prefetch starting for apps...', apps);
  }

  // 加载所有微应用的静态资源
  apps.forEach(({ entry }) => prefetch(entry, opts));
}
```