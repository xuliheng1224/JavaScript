## 前言
在上篇的[《JavaScript深入之执行上下文栈》](https://github.com/xuliheng1224/JavaScript/blob/master/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97/JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%E6%A0%88.md)中讲到，当JavaScript代码执行一段可执行代码时，会创建对应的执行上下文。

对于每个执行上下文，都有三个重要属性：
  
  * 变量对象(Variable object，VO)
  * 作用域链(Scope chain)
  * this

今天重点讲讲创建变量对象的过程。

## 变量对象
变量对象是与执行 上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明。

因为不同执行上下文下的变量对象稍有不同，所以我们来聊聊全局上下文下的变量对象和函数上下文下的变量对象。

## 全局上下文
我们先了解一个概念，叫全局对象。在 [W3School](http://www.w3school.com.cn/jsref/jsref_obj_global.asp) 中也有介绍：

>全局对象是预定义的对象，作为JavaScript的全局函数和全局属性的占位符。通过使用全局对象，可以访问所有其他所有预定义的对象、函数和属性。

>在顶层JavaScript代码中，可以用关键字this引用全局对象。因为全局对象是作用域链的头，这意味着所有非限定性的变量和函数名都会作为对象的属性来查询。

>例如，当JavaScript代码引用parseInt()函数时，他引用的是全局对象的parseInt属性。全局对象是作用域链的头，还意味着在顶层JavaScript代码中声明的所有变量都将成为全局对象的属性。

如果看的不是很懂的话，容我再来介绍下全局对象：

1.可以通过this引用，正在客户端JavaScript中，全局对象就是Window对象。

```js
console.log(this);
```

2.全局对象是由Object构造函数实例化的一个对象。

```js
console.log(this instanceof Object);
```

3.预定义了一堆，嗯，一大堆函数和属性。

```js
// 都能生效
console.lo(Math.random());
console.log(this.Math,random());
```

4.作为全局变量的宿主。

```js
var a = 1;
console.log(this.a);
```

5.客户端JavaScript中，全局对象有window属性指向自身。

```js
var a = 1;
console.log(window.a);

this.window.b = 2;
console.log(this.b);
```

花看一个大篇幅介绍全局对象，其实就想说：

全局上下文中的变量对象就是全局对象呐！

## 函数上下文
在函数上下文中，我们用活动对象(activation object,AO)来表示变量对象。

活动对象和变量对象其实是一个东西，只是变量对象是规范上的或者说是引擎上实现的，不可在JavaScript环境中访问，只有到当进入一个执行上下文中，这个执行上下文的变量对象才会被激活，
所以才叫活动变量，而只有被激活的变量对象，也就是活动变量上的各种属性才能被访问。

活动对象是在进入函数上下文时被创建的，它通过函数的arguments属性初始化。arguments属性值是Arguments对象。

## 执行过程
执行上下文的代码会分成两个阶段进行处理：分析和执行，我们也可以叫做：

  1.进入执行上下文
  2.代码执行

## 进入执行上下文
当进入执行上下文时，这时候还没有执行代码。

变量对象会包括：

  1.函数的所有形参（如果是函数上下文）

      。由名称和对应值组成的一个变量对象的属性被创建

      。没有实参，属性值设为undefined

  2.函数声明

      。由名称和对应值(函数对象(function-object))组成一个变量对象的属性被创建

      。如果变量对象已经存在相同名称的属性，则完全替换这个属性

  3.变量声明

      。由名称和对应值（undefined）组成一个变量对象的属性创建

      。如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性

举个例子：

```js
function foo(a) {
  var b = 2;
  function c() {}
  var d = fucntion() {};

  b = 3;

}

foo(1);
```

在进入执行上下文后，这时候的AO是：

```js
AO = {
  arguments:{
    0: 1,
    length: 1
  }
  a: 1,
  b: undefined,
  c: reference to funtion c() {},
  d: undefined
}
```

## 代码执行
在代码执行阶段，会顺序执行代码，根据代码，膝盖变量对象的值

还是上面的例子，当代码执行完后，这个时候的AO是：

```js
Ao = {
  arguments: {
    0: 1,
    length: 1
  },
  a: 1,
  b: 3,
  c: reference to function c() {},
  d: reference to FunctionExpression "d"
}
```

到这里变量对象的创建过程介绍完了，让我们简洁的总结我们上述所说：

  1.全局上下文的变量对象初始化是全局对象

  2.函数上下文的变量对象初始化只包括Arguments对象

  3.在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值

  4.在代码执行阶段，会再次修改变量对象的属性值

## 思考题
最后在让我们看几个例子：

1.第一题

```js
function foo() {
  console.log(a);
  a = 1;
}

foo(); // ?

function bar() {
  a = 1;
  console.log(a);
}
bar(); // ?
```

第一段会报错：`Uncaught ReferenceError: a is not defined`。

第二段会打印：1。

这是因为函数中的“a”并没有通过var关键字声明，所以不会被存放在AO中。

第一段执行console的时候，AO的值是：

```js
AO = {
  arguments:{
    length:0
  }
}
```

没有a的值，然后就会到全局去找，全局也没有，所以会报错。

当第二段执行console的时候，全局对象已经赋予了a属性，这时候就可以从全局找到a的值，所以会打印1。

2.第二题

```js
console.log(foo);

function foo() {
  console.log("foo");
}

var foo = 1;
```

会打印函数，而不是undefined。

这是因为在进入执行上下文时，首先会处理函数声明，如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性。

## 下一篇文章
[JavaScript深入之作用域链](https://github.com/xuliheng1224/JavaScript/blob/master/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97/JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E4%BD%9C%E7%94%A8%E5%9F%9F%E9%93%BE.md)









