# [JS数据类型、赋值、深拷贝和浅拷贝](https://github.com/monkeyWangs/blogs/issues/18)

## js 数据类型

1. 六种 基本数据类型:

- Boolean. 布尔值，true 和 false.
- null. 一个表明 null 值的特殊关键字。 JavaScript 是大小写敏感的，因此 null 与 Null、NULL或其他变量完全不同。
- undefined. 变量未定义时的属性。
- Number. 表示数字，例如： 42 或者 3.14159。
- String. 表示字符串，例如："Howdy"
- Symbol ( 在 ECMAScript 6 中新添加的类型).。一种数据类型，它的实例是唯一且不可改变的。

1. 以及 Object 对象引用数据类型

大多数情况下，我们可以通过`typeof`属性来判断。只不过有一些例外，比如:

```
var fn = new Function ('a', 'b', 'return a + b')

typeof fn // function
```

关于`function`属不属于js的数据类型，这里也有相关的讨论[JavaScript 里 Function 也算一种基本类型？](https://www.zhihu.com/question/24804474)

## 基本类型 和 引用数据类型 的相关区别

### 基本数据类型

我们来看一下 MDN 中对基本数据类型的一些定义：

> 除 Object 以外的所有类型都是不可变的（值本身无法被改变）。例如，与 C 语言不同，JavaScript 中字符串是不可变的（译注：如，JavaScript 中对字符串的操作一定返回了一个新字符串，原始字符串并没有被改变）。我们称这些类型的值为“原始值”。

```js
var a = 'string'
a[0] = 'a'
console.log(a)  // string
```

我们通常情况下都是对一个变量重新赋值，而不是改变基本数据类型的值。在 js 中是没有方法是可以改变布尔值和数字的。倒是有很多操作字符串的方法，但是这些方法都是返回一个新的字符串，并没有改变其原有的数据。比如:

- 获取一个字符串的子串可通过选择个别字母或者使用 String.substr().
- 两个字符串的连接使用连接操作符 (+) 或者 String.concat().

### 引用数据类型

引用类型（object）是存放在堆内存中的，变量实际上是一个存放在栈内存的指针，这个指针指向堆内存中的地址。每个空间大小不一样，要根据情况开进行特定的分配，例如。

```js
var person1 = {name:'jozo'};
var person2 = {name:'xiaom'};
var person3 = {name:'xiaoq'};
```

[![img](https://camo.githubusercontent.com/ee34eea35e4899d359750c7d4fe52eaf2db96ce993e1e8c03bb577757a882017/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f332f36666232633364313364383330656663366165303761633336386466303831363f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)](https://camo.githubusercontent.com/ee34eea35e4899d359750c7d4fe52eaf2db96ce993e1e8c03bb577757a882017/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f332f36666232633364313364383330656663366165303761633336386466303831363f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)

引用类型的值是可变的：

```js
person1['name'] = 'muwoo'

console.log(person1) // {name: 'muwoo'}
```

### 传值与传址

了解了基本数据类型与引用类型的区别之后，我们就应该能明白传值与传址的区别了。
在我们进行赋值操作的时候，基本数据类型的赋值（=）是在内存中新开辟一段栈内存，然后再把再将值赋值到新的栈中。例如:

```js
var a = 10;
var b = a;

a ++ ;
console.log(a); // 11
console.log(b); // 10
```

[![img](https://camo.githubusercontent.com/2562af8fb0b4a89d6930883e5ab8217b1a500df9e5ca8000a2fc3ece9a6b50b1/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f332f38643937336139373138646131383036643139646230633135343166663432353f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)](https://camo.githubusercontent.com/2562af8fb0b4a89d6930883e5ab8217b1a500df9e5ca8000a2fc3ece9a6b50b1/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f332f38643937336139373138646131383036643139646230633135343166663432353f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)

所以说，基本类型的赋值的两个变量是两个独立相互不影响的变量。

但是引用类型的赋值是传址。只是改变指针的指向，例如，也就是说引用类型的赋值是对象保存在栈中的地址的赋值，这样的话两个变量就指向同一个对象，因此两者之间操作互相有影响。例如：

```js
var a = {}; // a保存了一个空对象的实例
var b = a;  // a和b都指向了这个空对象

a.name = 'jozo';
console.log(a.name); // 'jozo'
console.log(b.name); // 'jozo'

b.age = 22;
console.log(b.age);// 22
console.log(a.age);// 22

console.log(a == b);// true
```

[![img](https://camo.githubusercontent.com/148a24bdb3033fbe698c8d3183d3cce404049996e57084155ff30c392e0eebc9/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f332f30316461643964633030666230656665383164396263626539643330613162643f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)](https://camo.githubusercontent.com/148a24bdb3033fbe698c8d3183d3cce404049996e57084155ff30c392e0eebc9/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f332f30316461643964633030666230656665383164396263626539643330613162643f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)

## 浅拷贝

先来看一段代码的执行：

```js
var obj = {a: 1, b: {c: 2}}
var obj1 = obj
var obj2 = shallowCopy(obj);
function shallowCopy(src) {
    var dst = {};
     for (var prop in src) {
         if (src.hasOwnProperty(prop)) {
             dst[prop] = src[prop];
          }
      }
     return dst;
}

var obj3 = Object.assign({}, obj)

obj.a = 2
obj.b.c = 3

console.log(obj) // {a: 2, b: {c: 3}}
console.log(obj1) // {a: 2, b: {c: 3}}
console.log(obj2) // {a: 1, b: {c: 3}}
console.log(obj3) // {a: 1, b: {c: 3}}
```

这段代码可以说明赋值得到的对象 `obj1` 只是将指针改变，其引用的仍然是同一个对象，而浅拷贝得到的的 `obj2` 则是重新创建了新对象。但是，如果原对象`obj`中存在另一个对象，则不会对对象做另一次拷贝，而是只复制其变量对象的地址。这是因为浅拷贝只复制一层对象的属性，并不包括对象里面的为引用类型的数据。
对于数组，更长见的浅拷贝方法便是`slice(0)`和 `concat()`
ES6 比较常见的浅拷贝方法便是 `Object.assign`

## 深拷贝

通过上面的这些说明，相信你对深拷贝大致了解了是怎样一个东西了：深拷贝是对对象以及对象的所有子对象进行拷贝。那么如何实现这样一个深拷贝呢？

### 1. JSON.parse(JSON.stringify(obj))

对于常规的对象，我们可以通过`JSON.stringify`来讲对象转成一个字符串，然后在用`JSON.parse`来为其分配另一个存储地址，这样可以解决内存地址指向同一个的问题：

```js
var obj = {a: {b: 1}}
var copy = JSON.parse(JSON.stringify(obj))

obj.a.b = 2
console.log(obj) // {a: {b: 2}}
console.log(copy) // {a: {b: 1}}
```

但是 `JSON.parse()`、`JSON.stringify`也存在一个问题，`JSON.parse() `和J `SON.stringify()`能正确处理的对象只有`Number、String、Array`等能够被 json 表示的数据结构，因此函数这种不能被 json 表示的类型将不能被正确处理。

```js
var target = {
    a: 1,
    b: 2,
    hello: function() { 
            console.log("Hello, world!");
    }
};
var copy = JSON.parse(JSON.stringify(target));
console.log(copy);   // {a: 1, b: 2}
console.log(JSON.stringify(target)); // "{"a":1,"b":2}"
```

### 2. 遍历实现属性复制

既然浅拷贝只能实现非`object`第一层属性的复制，那么遇到`object`只需要通过递归实现浅拷贝其中内部的属性即可：

```js
function extend (source) {
  var target
  if (typeof source === 'object') {
    target = Array.isArray(source) ? [] : {}
    for (var key in source) {
      if (source.hasOwnProperty(key)) {
        if (typeof source[key] !== 'object') {
          target[key] = source[key]
        } else {
          target[key] = extend(source[key])
        }
      }
    }
  } else {
    target = source
  }
  return target
}

var obj1 = {a: {b: 1}}
var cpObj1 = extend(obj1)
obj1.a.b = 2
console.log(cpObj1) // {a: {b: 1}}

var obj2 = [[1]]
var cpObj2 = extend(obj2) 
obj2[0][0] = 2
console.log(cpObj2) // [[1]]
```

我们再来看一下 `Zepto` 中深拷贝的代码:

```js
    // 内部方法：用户合并一个或多个对象到第一个对象
    // 参数：
    // target 目标对象  对象都合并到target里
    // source 合并对象
    // deep 是否执行深度合并
    function extend(target, source, deep) {
        for (key in source)
            if (deep && (isPlainObject(source[key]) || isArray(source[key]))) {
                // source[key] 是对象，而 target[key] 不是对象， 则 target[key] = {} 初始化一下，否则递归会出错的
                if (isPlainObject(source[key]) && !isPlainObject(target[key]))
                    target[key] = {}

                // source[key] 是数组，而 target[key] 不是数组，则 target[key] = [] 初始化一下，否则递归会出错的
                if (isArray(source[key]) && !isArray(target[key]))
                    target[key] = []
                // 执行递归
                extend(target[key], source[key], deep)
            }
            // 不满足以上条件，说明 source[key] 是一般的值类型，直接赋值给 target 就是了
            else if (source[key] !== undefined) target[key] = source[key]
    }
```

内部实现其实也是差不多。

## 参考资料

[js 深拷贝 vs 浅拷贝](https://juejin.im/post/59ac1c4ef265da248e75892b)