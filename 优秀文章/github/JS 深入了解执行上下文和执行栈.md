# JS 深入了解执行上下文和执行栈

## 执行上下文（Execution Context）

在运行一段 js 代码的时候，就会生成一个可执行的上下文，根据不同类型的区分，可能会存在以下几种类型的代码执行上下文：

- Global code - 代码第一时间指的全局环境
- Function code - 函数代码块
- Eval code - eval 函数包含的代码块

我们来看一段代码：

[![img](https://camo.githubusercontent.com/c90f631b9bfc4081c4fac158a7fbe1b9d077ce7e412d81352717b153637cfb15/687474703a2f2f6461766964736861726966662e636f6d2f626c6f672f77702d636f6e74656e742f75706c6f6164732f323031322f30362f696d67312e6a7067)](https://camo.githubusercontent.com/c90f631b9bfc4081c4fac158a7fbe1b9d077ce7e412d81352717b153637cfb15/687474703a2f2f6461766964736861726966662e636f6d2f626c6f672f77702d636f6e74656e742f75706c6f6164732f323031322f30362f696d67312e6a7067)

这段代码中，我们存在一个 `global context` 和 3 个不同的 `function context`。你可以随意创建这样的`function context`但是每一个函数都会创建一个新的执行上下文（EC）。函数外部定义的变量可以被函数内部使用，但是函数内部定义的变量确不能被函数外部使用，这是为什么呢？

## 执行栈（Execution Context Stack）

浏览器解释器执行 js 是单线程的过程，这就意味着同一时间，只能有一个事情在进行。其他的活动和事件只能排队等候，生成出一个等候队列执行栈（Execution Stack）：
[![img](https://camo.githubusercontent.com/45283cd82767e3b7714cf6e60870ba5faccce15e4962ca88992f7e2a1f88ba2a/687474703a2f2f6461766964736861726966662e636f6d2f626c6f672f77702d636f6e74656e742f75706c6f6164732f323031322f30362f6563737461636b2e6a7067)](https://camo.githubusercontent.com/45283cd82767e3b7714cf6e60870ba5faccce15e4962ca88992f7e2a1f88ba2a/687474703a2f2f6461766964736861726966662e636f6d2f626c6f672f77702d636f6e74656e742f75706c6f6164732f323031322f30362f6563737461636b2e6a7067)

我们知道，在一开始执行代码的时候，变确定了一个全局执行上下文`global execution context`作为默认值。如果在你的全局环境中，调用了其他的函数，程序将会再创建一个新的 EC，然后将此 EC推入进执行栈中`execution stack`

如果函数内再调用其他函数，相同的步骤将会再次发生：创建一个新的EC -> 把EC推入执行栈。一旦一个EC执行完成，变回从执行栈中推出（pop）：

```js
(function foo(i) {
    if (i === 3) {
        return;
    }
    else {
        foo(++i);
    }
}(0));
```

上面这个函数完成了对自身的3次调用，第一次调用时，生成一个EC推入执行栈，在执行的过程中，又遇到一个新的函数`foo(1)`此时，又会创建一个新的EC，再次推入...

关于执行栈，有5个关键点可以了解一下：

- 单线程
- 同步执行
- 1 个全局执行上下文 （ Global context）
- 无线多的 function context
- 每个函数调用都会创建一个新的 EC，即使是调用自身

## 执行上下文（EC）的一些细节

从上文了解到了一旦函数被调用，都会创建一个新的执行上下文。但是在 js 解释器中，调用执行上下文有2个阶段：

1. 创建阶段（函数被调用，但是还没有执行）：

- 创建作用域链 `scope chain`
- 定义arguments、变量、函数
- 确定 this 值

1. 激活/代码执行阶段

- 主要是对之前定义的变量分配值，开始解释执行代码。

当每一个 function 被调用时，我们可以大致描述一下会创建这样一个`executionContextObj`对象

```js
executionContextObj = {
    'scopeChain': { /* 变量对象 + 父执行上下文上的变量对象 */ },
    'variableObject': { /* function arguments /  parameters，内部的 variable 和 function 声明 */ },
    'this': {}
}
```

## 活动对象和变量对象（Activation / Variable Object [AO/VO]）

`executionContextObj`在函数被调用的时候创建，在创建阶段，会扫描函数的所有参数和变量。扫描的结果成了`executionContextObj`内部的`variableObject`属性。下面是执行步骤的细化：

1. 发现了一个函数调用
2. 在执行函数执行，创建了一个 EC，推入执行栈
3. 进入执行阶段：

- 初始化作用域链
- 创建 `variableObject`：内部包含 arguments 对象；内部定义的函数，以及绑定上对应的变量环境；内部定义的变量

1. 激活阶段：运行函数内部的代码，对变量复制，代码一行一行的被解释执行.

可以通过一段代码来分析一下：

```js
function foo(i) {
    var a = 'hello';
    var b = function privateB() {

    };
    function c() {

    }
}

foo(22);
```

当我们执行`foo(22)`的时候，EC创建阶段会类似生成下面这样的对象：

```js
fooExecutionContext = {
    scopeChain: { ... },
    variableObject: {
        arguments: {
            0: 22,
            length: 1
        },
        i: 22,
        c: pointer to function c()
        a: undefined,
        b: undefined
    },
    this: { ... }
}
```

正如你看到的，在创建阶段，会发生属性名称的定义，但是并没有赋值。一旦创建阶段（creation stage）结束，变进入了激活 / 执行阶段，那么`fooExecutionContext`便会完成赋值，变成这样：

```js
fooExecutionContext = {
    scopeChain: { ... },
    variableObject: {
        arguments: {
            0: 22,
            length: 1
        },
        i: 22,
        c: pointer to function c()
        a: 'hello',
        b: pointer to function privateB()
    },
    this: { ... }
}
```