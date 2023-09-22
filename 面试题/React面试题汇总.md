# React面试题汇总

如果你是一位有抱负的前端程序员并准备面试，那么这篇文章很适合你。本文是你学习和面试 React 所需知识的完美指南。

### 1、**说说react里面bind与箭头函数**

- **bind 由于在类中,采用的是**

`严格模式`,所以事件回调的时候

`会丢失this指向,指向undefined`,需要使用bind来给函数绑定上当前实例的this指向

- **箭头函数的this指向上下文,所以永久能拿到当前组件实例的**

`this`指向,我们可以完美的使用箭头函数来替代传统事件处理函数的回调

### **2、说说react中的性能优化**

性能优化分为2个方面

1. 使用shouldComponentUpdate来对state和props进行对比,如果两次的结果一直,那么就return false
2. 使用纯净组件,pureComponent

### 3、**高阶组件和高阶函数是什么**

1. 高阶函数:函数接收一个函数作为参数,或者将函数作为返回值的函数就是高阶函数 map some every filter reduce find forEach等等都属于高阶函数
2. 高阶组价:接受一个组件,返回一个新组建的组件就是高阶组件,本质上和高阶函数的意思一样
3. 高阶组件是用来复用react代码的一种方式

### **4、setState和repalceState的区别**

setState 是修改其中的部分状态，相当于 Object. assign，只是覆盖，不会减少原来的状态；

replaceState 是完全替换原来的状态，相当于赋值，将原来的 state 替换为另一个对象，如果新状态属性减少，那么 state 中就没有这个状态了

### **5、redux中核心组件有哪些，reducer的作用**

**redux 三大核心**

- action action理解为动作，action的值一般为一个对象，格式如 { type: "", data: "" }，type是必须要的，因为reducer处理数据的时候要根据不同的type来进行不同的操作
- reducer reducer是初始化以及处理派发的action的纯函数
- store store是一个仓库，用来存储数据，它可以获取数据，也可以派发数据，还能监听到数据的变化

**reducer的作用**

接收旧的 state 和 action，返回新的 state

### **6、什么是受控组件**

受控组件就是可以被 react 状态控制的组件

在 react 中，Input textarea 等组件默认是非受控组件（输入框内部的值是用户控制，和React无关）。但是也可以转化成受控组件，就是通过 onChange 事件获取当前输入内容，将当前输入内容作为 value 传入，此时就成为受控组件。

### **7、hooks+context和redux你是怎么选择的，都在什么场景下使用？**

如果项目体量较小，只是需要一个公共的store存储state，而不讲究使用action来管理state，那context完全可以胜任。反之，则是redux的优点。

**使用场景:**

如果你在组件间传递的数据逻辑比较复杂，可以使用redux；

如果组件层级不多，可以使用props；

如果层级较深，数据逻辑简单，可以使用context。

### **8、useffect 模拟生命周期**

- **模拟componentDidMount**

第二个参数为一个空数组，可以模拟componentDidMount

```js
componentDidMount：useEffect(()=>{console.log('第一次渲染时调用')},[]) 
```

- **模拟componentDidUpdate**

没有第二个参数代表监听所有的属性更新

```JS
useEffect(()=>{console.log('任意状态改变')})  
```

监听多个属性的变化需要将属性作为数组传入第二个参数。

```JS
useEffect(()=>{console.log('指定状态改变')},[状态1,状态2...])  
```

- **模拟componentWillUnmount**

```JS
useEffect(()=>{   ...  return()=>{ //组件卸载前}   }) 
```

### **9、setsate更新之后和usestate的区别**

- **setState( updater [,callback] )**

updater：object/function - 用于更新数据

callback：function - 用于获取更新后最新的 state 值

构造函数是唯一建议给 this.state 赋值的地方

不建议直接修改 state 的值，因为这样不会重新渲染组件

自动进行浅合并（只会合并第1层）

由于 setState() 异步更新的缘故，依赖 state 旧值更新 state 的时候建议采用传入函数的方式

- **useState(initState)**

const [ state , setState ] = useState(initState)

state：状态

setState(updater) ：修改状态的方法

updater：object/function - 用于更新数据

initState：状态的初始值

### **10、react父组件props变化的时候子组件怎么监听**？

当props发生变化时执行，初始化render时不执行，在这个回调函数里面，你可以根据属性的变化，通过调用this.setState()来更新你的组件状态，旧的属性还是可以通过this.props来获取,这里调用更新状态是安全的，并不会触发额外的render调用

```js
//props发生变化时触发 
componentWillReceiveProps(props) { 
        console.log(props) 
    this.setState({show: props.checked}) 
} 
```

### **11、usememo在react中怎么使用？**

返回一个 memoized 值。

把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。

传入 useMemo 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 useEffect 的适用范畴，而不是 useMemo。

如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。

你可以把 useMemo 作为性能优化的手段，但不要把它当成语义上的保证。将来，React 可能会选择“遗忘”以前的一些 memoized 值，并在下次渲染时重新计算它们，比如为离屏组件释放内存。先编写在没有 useMemo 的情况下也可以执行的代码 —— 之后再在你的代码中添加 useMemo，以达到优化性能的目的。

### **12、React Component和Purecomponent区别**

Component 没有直接实现 shouldComponentUpdate 这个方法；但是 PureComponent通过浅层的Porps 和 state 的对比，内部实现了这个生命周期函数。

PureComponent会跳过整个组件子树的props更新，要确保全部的子组件也是 pure 的形式。

Component 中需要手动执行的 shouldComponentUpdate 函数，在PureComponent中已经自动完成了（自动浅对比）。

PureComponent不仅会影响本身，而且会影响子组件，所以PureComponent最好用在数据展示组件中

PureCoponent 如果是复杂数据类型，这里会造成错误的显示（setState浅复制更新，但是界面不会重新渲染）

### 13、**hooks相对于class的优化**

**类组件缺点一：复杂且不容易理解的“this”**

Hooks解决方式：函数组件和普通JS函数非常相似，在普通JS函数中定义的变量、方法都可以不使用“this.”，而直接使用该变量或函数，因此你不再需要去关心“this”了。

**类组件缺点二：组件数据状态逻辑不能重用**

Hooks解决方式：

通过自定义Hook，可以数据状态逻辑从组件中抽离出去，这样同一个Hook可以被多个组件使用，解决组件数据状态逻辑并不能重用的问题。

**类组件缺点三：组件之间传值过程复杂、缺点三：复杂场景下代码难以组织在一起**

Hooks解决方式：

通过React内置的useState()函数，可以将不同数据分别从"this.state"中独立拆分出去。降低数据复杂度和可维护性，同时解决类组件缺点三中“内部state数据只能是整体，无法被拆分更细”的问题。

通过React内置的useEffect()函数，将componentDidMount、componentDidUpdate、componentWillUncount 3个生命周期函数通过Hook(钩子)关联成1个处理函数，解决事件订阅分散在多个生命周期函数的问题。

### 14、**hooks父组件怎么调用子组件的方法？**

父组件使用 `useRef` 创建一个 `ref` 传入 子组件 子组件需要使用

`useImperativeHandle` 暴露 `ref` 自定义的实例值给父组件。这个需要用 `forwardRef` 包裹着。

### 15、**react通过什么方法修改参数？**

类组件修改数据的方法, 通过setState , 注意setState的修改方法有两种，而且它是异步的

函数组件修改方式通过自定义的方法。需要通过 useState ， 例如。const [count,setCount] = useState(0)

这里面setCount 可以修改count参数

### 16、**说你对react native的了解**

React native 基于 JavaScript 开发的一个 可以开发原生app的这么一个集成框架，它兼容开发 iOS 和 Android能够实现一套代码，两个系统都能使用，方便维护，相比于web前端的 react ，react-native更好的提供了一些调用手机硬件的API,可以更好的开发移动端，现在react-native它的生态环境也是越来越好，基本上可以完全实现原生开发，

但是现在好多的应用还是用它来套壳 （原生 + web前端），用它来做有些路由，和框架的搭建，然后里面内容来使用前端的react来实现，这样一来让维护更方便，开发更快捷

### 17、**react里的一个输入框每当用户输入文字就会触发onchange，我们怎么拿到他最后输入完的结果**

1.第一种是通过受控组件，可以获取到state里面的值，获取修改结果

```scala
        class Home extends React.Component { 
                state = { 
                        val:"" 
                } 
                //这是一个通用的写法，然后注意 name的值一定要与state定义的一直 
                changeInput = (e) =>{ 
                        let {name,value} = e.target 
                        this.setState({ 
                                [name]:value 
                        }) 
                } 
                render(){ 
                        return <> 
                                <input onChange="this.state.val" /> 
                        </> 
                } 
        } 
 
```

2.第二种通过ref来获取里面的值

```scala
class Home extends React.Component { 
         render(){ 
                 //          this.val 获取里面的真是dom 
                         return <input ref={node=>this.val} /> 
         } 
} 
```

### 18、**react的render什么时候渲染？**

react生命周期有三个阶段。 两个阶段都会执行 render 主要从更新和挂载两个阶段来讲，挂载阶段都顺序，更新阶段一定要说  shouldComponentUpdate true 和    false    分别对应后边是否执行render

1）挂载阶段

```js
      constructor(){} 
 
       static getDerivedStateFromProps(){ 
 
              return {} 
 
       } 
 
       render(){}.       挂载阶段会执行一次 
 
       componentDidMount(){} 
```

2 ） 更新阶段

```js
       static getDerivedStateFromProps(props, state){rerturn {}} 
 
        shouldComponentUpdate(nextProps, nextState).{ return Boolean. 注意如果 false 则 不向下执行 ，true的时候会执行render} 
 
        return。true  
 
        render() ..                 
```

### 19、**useEffect的依赖为引用类型如何处理**

useEffect的依赖为引用类型的时候，可能会导致监听不触发，原因就是监听的同一个地址的时候，对象本身地址没变，所以监听的结果就是认为数据并没有改变从而不直接调用

解决方案

1.如果数据是对象的话，可以监听对象的里面的值，值是基本类型，如果值改变了，那么可以监听执行

2.在去修改对象和数据的时候，使用深拷贝或者浅拷贝，这样地址发生改变可以监听执行

3.可以转成字符串，通过JSON.stringify() ,监听字符串这样的，这样改变也会执行

### 20、**react组件的key说一下 key发生变化会发生什么 key 值不同销毁前会触发什么生命周期**

1.react的key值给组件作为唯一标识，类似身份证，当数组发生增删改查的时候可以通过这个 标识来去对比虚拟dom的更改前后，如果相同标识的数据或者属性没有改变的话，讲直接略过，对比有改变的然后直接改变此项

2.如果给组件添加key,如果key值改变的时候，组件将会重新出创建会执行 挂在阶段的生命周期

constructor

static getDerivedStateFromProps()

render(){}

componentDidUpdate



### 21、**知道react里面的createPortal么？说说使用场景。之前react没有这个的时候是怎么实现的，有他没他的区别**

react.createPortal 这个方法是来使用做弹窗Modal组件的,在没有这个组件之前我们可以自己定义组件，然后去实现Modal效果

```js
        Modal.js 
const styles = { 
  modal: { 
    position: 'fixed', 
    top: 0, 
    left: 0, 
    right: 0, 
    bottom: 0, 
    backgroundColor: 'rgba(0,0,0,0.3)', 
    display: 'flex', 
    justifyContent: 'center', 
    alignItems: 'center' 
  } 
} 
 
class Modal extends Component { 
  constructor(props) { 
    super(props); 
  } 
  render() { 
    return ( 
      <div style={styles.modal}> 
        {this.props.children} 
      </div> 
    ); 
  } 
} 
```

`react.createPortal`   这个来制作弹窗组件，它 在Modal 组件位置进行 `fixed` 定位，可以任意的挂载到某个dom元素上，使用后的管理更方便，但是注意需要预留html的挂载元素。



### 22、**react全家桶**

真正意义上的react全家桶，其实指的是react，react-dom，react-native。

因为react核心库只是vDom的操作，无关dom(dom操作在react-dom中)——这意味着它天生就可以做到跨平台。

注意这里有误区，react-router，react-router-dom，react-dux只是社区的一些使用较多的解决方案，事实上它们各有缺陷。



### 23、**如何理解fiber？**

截止到当前版本(react17.0.2),react尚存在一个明显的缺陷——这是大多数vDom框架都需要面对的——“diff”。

要知道，render前后的两颗vDom tree进行diff，这个过程是不可中断的(以tree为单位，不可中断)，这将造成当diff的两颗tree足够庞大的时候，js线程会被diff阻塞，无法对并发事件作出回应。

为了解决这个问题，react将vDom节点上添加了链表节点的特性，将其改造成了fiber节点（其实就是vdom节点结合了链表节点的特性），目的是为了后面的Fiber架构的实现，以实现应对并发事件的“并发模式”。

事实上，原计划是17版本上的，但最终定期在了18版本。

### 24、**时间分片，任务分炼**

fiber节点的改造是为了将diff的单位从不可中断的tree降级到可中断的单一vDom。这意味着每次一个vDom节点比对过后，可以轮训是否发生了优先级更高的并发事件(react将用户想更快得到反馈的事进行了优先级排序，比如js动画等)。

每个vDom的比对是一个时间片单位，因此基于fiber节点的Fiber架构，存在这样的特性：

“时间分片，任务分炼，异步渲染。”

### 25、**你了解hooks吗？谈谈你对hooks的理解？聊聊hooks？**

react-hooks提供给了  `函数式组件  以业务逻辑抽离封装的能力`  ，从单一组件单位进行UI与业务逻辑的分离，以此来保证函数式组件的UI纯净。

在此之前，我们只能通过状态提升的方式，将业务逻辑提升到容器组件的方式。

### **26、hooks使用规则？**

- 只可以在顶层调用栈调用hooks

不可以在循环/条件/嵌套里调用hooks

- 只可以在react function或自定义hooks里调用hooks

原因见hooks调用记录表底层

### 27、**hooks调用记录表底层_单链表结构**

hooks的引用记录，是用单链表结构实现的；(可在力扣/leetcode网找一下实现单链表的算法)

每一个链表节点(node)都有一个next的指针指向下一个node；

如果中间有缺失，那么就无法找到下一个next；

因此hooks不可以在条件里调用（有可能丢失链表node，导致hooks遍历中断）；

### 28、**useEffect与副作用？什么是副作用？聊聊useEffect？**

- **副作用函数**

我们将跟UI渲染无关的业务逻辑称之为副作用。

useEffect是react在函数式组件里提供的副作用解决方案，它接受两个参数。

第一个是必传参数，类型为函数。

我们写在此函数中的业务逻辑代码就是我们所说的副作用。

- **消除副作用函数**

默认情况下，useEffct会在每次render后执行传入的副作用函数。

然而有的时候我们需要在执行副作用前先去除掉上次render添加的副作用效果,我们可以在副作用函数里再返回一个函数———这个函数就是消除副作用，它会在每次reRender前和组件卸载时去执行。

- **监听器**

与 useMemo, useCallback等一样，useEffect可以传入一个监听数组参数，这意味着副作用只有在数组中的监听值变化时才会执行。

我们借助它的这个参数特性，可以模拟类组件生命周期钩子————比如componentDidMount等。

### 29、**生命周期模拟？**

useEffect的本质作用，其实是将函数式组件中的业务逻辑副作用进行抽离封装。模拟生命周期并不是它的核心思想，但它当然能做到这个事：

[https://github.com/melodyWxy/melody-core-utils/blob/master/src/lifeHooks/index.ts](https://link.zhihu.com/?target=https%3A//github.com/melodyWxy/melody-core-utils/blob/master/src/lifeHooks/index.ts)

### 

### **30、useLayoutEffect？**

与useEffect的区别是，useLayoutEffect中副作用的执行时间在组件render之前，这意味着它适用于比组件render更紧急的副作用。

正因为如此，多数情况下我们不会选择使用它，因为我们并不想阻塞render。

希望这套React面试题和答案能帮你准备面试。祝一切顺利！