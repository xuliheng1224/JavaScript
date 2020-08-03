
## 前言
在上篇的[《JavaScript深入之执行上下文栈》](https://github.com/xuliheng1224/JavaScript/blob/master/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97/JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%E6%A0%88.md)中讲到，当JavaScript代码执行一段可执行代码时，会创建对应的执行上下文。

对于每个执行上下文，都有三个重要属性：
  
  * 变量对象(Variable object，VO)
  * 作用域链(Scope chain)
  * this

今天重点讲讲作用域链。

## 作用域链
在《JavaScript深入之变量对象》中讲到，当查找变量的时候，会先从当前上下午文的变量对象中查找，如果没有找到，就会从父级执行上下文的变量对象中查找，
一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做作用域链。

下满，让我们以一个函数的创建和激动两个时期来讲解作用域链是如何创建和变化的。

## 函数创建
在《JavaScript深入之词法作用域和动态作用域》中讲到，函数的作用域在函数定义的时候就决定了。

这是因为函数有一个内部属性[[scope]]，当函数创建的时候，就会保存所有父变量对象到其中，你可以理解[[scope]]就是所有父变量对象的层级链，但是注意：
[[scope]]并不代表完整的作用域链！

举个例子：

```js
function foo() {
  function bar(){
    ...
  }
}
```

函数创建时，各自的[[scope]]为：

```js
foo.[[scope]] = [
  globalContext.VO
];

bar.[[scope]] = [
  fooContext.AO,
  globalConetxt.VO
]
```

## 函数激活
当函数激动时，进入函数上下文，创建VO/AO后，就会将活动对象添加到作用域的前端。

这个时候执行上线的作用域链，我们命名为Scope：

```js
Scope = [AO].concat([[scope]]);
```

至此，作用域链创建完毕。

## 捋一捋
以下面的例子为例，结合着之前将的变量对象和执行上下文栈，我们来总结一下函数执行上下文中作用域链和变量对象的创建过程：

```js
var scope = "global scope";
function checkscope() {
  var scope2 = 'local scope';
  return scope2;
}
checkscope();
```

执行过程如下：

1.checkscope函数被创建，保存作用域到内部属性[[scope]]

```js
checkscope.[[scope]] = [
  globalContext.VO
]
```

2.执行checkscope函数，创建checkscope函数执行上下文，checkscope函数执行上下文被压入执行上下文栈

```js
ECStack = [
  checkscopeContext,
  globalContext
];
```

3.checkscope函数并不立刻执行，开始做准备工作，第一步：复制子函数[[scope]]属性创建作用域链

```js
checkscopeContext = {
  Scope: checkscope.[[scope]]
}
```

4.第二步：用arguments创建活动对象，随后初始化活动对象，加入形参、函数声明、变量声明

```js
checkscopeConetext = {
  AO: {
    arguments:{
      length: 0
    },
    scope2: undefined
  },
  Scope: checkscope.[[scope]]
}
```

5.第三步：将活动对象压入checkscope作用域顶端

```js
checkscopeContext = {
  AO: {
    arguments: {
      length: 0,
    },
    scope2: undefined
  },
  Scope: [AO,[[scope]]]
}
```

6.准备工作做完，开始执行函数，随着函数的执行，修改AO的属性值

```js
checkscopeContext = {
  AO: {
    arguments: {
      length: 2,
    },
    scope2: 'local scope'
  },
  Scope: [AO,[[scope]]]
}
```

7.查找到scope2的值返回后函数执行完毕，函数上下文从执行上下文栈中弹出

```js
ECStack = [
  globalConetxt
];
```

## 下一篇文章
[JavaScript深入之从ECMAScript规范解读this](https://github.com/xuliheng1224/JavaScript/blob/master/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97/JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E4%BB%8EECMAScript%E8%A7%84%E8%8C%83%E8%A7%A3%E8%AF%BBthis.md)
