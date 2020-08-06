## 前言
在《JavaScript深入之执行上下文栈》中讲到，当JavaScript代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。

对于每个执行上下文，都有三个重要属性

  * 变量对象(Variable object，VO)
  * 作用域链(Scope chain)
  * this

今天重点讲讲 this

我们要从 ECMASciript5 规范开始讲起。

先奉上 ECMAScript 5.1 规范地址：

英文版：[http://es5.github.io/#x15.1](http://es5.github.io/#x15.1)

中文版：[http://yanhaijing.com/es5/#115](http://yanhaijing.com/es5/#115)

让我们开始了解规范吧！

## Types
首先是第8章的Types：

ECMAScript的类型分为语言类型和规范类型。

ECMAScript语言类型是开发者直接使用ECMAScript可以操作的。其实就是我们常说的undefined，null，Boolean，String，Number和Object。

而规范类型相当于meta-values,是用来用算法描述ECMAScript语言结构和语言类型的。规范类型包括：Reference，list，Completion，Property Descriptor，Property Identifier, Lexical Environment, 和 Environment Record。

我们只要知道在ECMAScript规范中还有一种只存在于规范中的类型，他们的作用是用来描述语言底层行为逻辑。

今天我们要讲的重点是其中的Reference类型。它与this的指向有着密切的关联。

## Reference
那什么又是Reference？

让我们看8.7章The Reference Specification Type，

所以Reference类型就是用来解释诸如delete、typeof以及赋值等操作行为的。

这段讲述了 Reference 的构成，由三个组成部分，分别是：

  * base value
  * referenced name
  * strict reference

可是这些到底是什么呢？

我们简单的理解话：

base value就是属性所在的对象或者就是EnvironmentRecord，它的值可能是undefined，an Object，a Boolean,a String,a Number,or an environment record其中一种。

reference name 就是属性的名称。

局个例子：

```js
var foo = 1;

// 对应的Reference是：
var fooReference = {
  base: EnvironmentRecord,
  name: 'foo',
  strict: false
};
```

在举一个例子

```js
var foo = {
  bar: function () {
    return this;
  }
};

foo.bar(); // foo

// bar 对应的Reference是：
var barReference = {
  base: foo,
  propertyName: 'bar',
  strict: false
};
```

而且规范中还提供了获取Reference组成部分的方法，比如GetBase和IsPropertyReference。

这两个方法很简单，简单看一看：

1.GetBase(v) 返回Reference的base value。

2.IsPropertyReference(v) 如果base value是一个对象，就返回true。

## GetValue
除此之外，紧接着在8.7.1章规范中就讲了一个用于从Reference类型获取对应值的方法：GetValue。

简单模拟GetValue的使用：

```js
var foo = 1;

var fooReference = {
  base: EnvironmentRecord,
  name: 'foo',
  strict: false
};

GetValue(fooReference); // 1
```

GetValue返回对象属性真正的值，但是要注意：

调用GetValue，返回的将是具体的值，而不再是一个Reference

## 如何确定this的值
关于Reference讲了那么多，为什么要讲Reference呢？到底Reference跟本文的主体this有哪些关联呢？

看规范11.2.3 函数调用:

这里讲当函数调用的时候，如何确定this的值。

只看第一步，第六步，第七步：

>1.令 ref 为解释执行 MemberExpression 的结果。

>6.如果 Type(ref) 为 Reference，那么 如果 IsPropertyReference(ref) 为 true，那么 令 thisValue 为 GetBase(ref). 否则 , ref 的基值是一个环境记录项 令 thisValue 为调用 GetBase(ref) 的 ImplicitThisValue 具体方法的结果

>7.否则 , 假如 Type(ref) 不是 Reference. 令 thisValue 为 undefined。

简单描述一下：

1.计算 MemberExpression 的结果赋值给 ref。

2.判断ref 是不是一个Reference类型

  >2.1如果 ref 是 Reference，并且 IsPropertyReference(ref) 为 true，那么 this 的值为 GetBase(ref)

  >2.2如果 ref 是 Reference，并且 base value 值是 Environment Record，那么 this 的值为 ImlicitThisValue(ref)

  >2.3如果 ref 不是 Reference，那么 this 的值为 undefined

## 具体分析
让我们一步一步看：

  1.计算 MemberExpression 的结果赋值给ref

什么是 MemberExpression？看规范11.2 左值表达式

MemberExpression :

* PrimaryExpression // 原始表达式 可以参见《JavaScript权威指南第四章》
* FunctionExpression    // 函数定义表达式
* MemberExpression [ Expression ] // 属性访问表达式
* MemberExpression . IdentifierName // 属性访问表达式
* new MemberExpression Arguments    // 对象创建表达式

举个例子：

```js
function foo() {
  console.log(this);
}

foo(); // MemberExpression 是 foo

function foo() {
  return function (){
    console.log(this);
  }
}

foo()(); // MemberExpression 是 foo()

var foo = {
  bar: function (){
    return this
  }
}

foo.bar();// MemberExpression 是 foo.bar
```

所以简单理解 MemberExpression 其实就是（）左边的部分。

2.判断ref是不是一个Reference类型。

关键在于看规范是如何处理各种MemberExpression，返回的结果是不是一个Reference类型。

举例：

```js
var value = 1;

var foo = {
  value: 2,
  bar: function() {
    return this.value;
  }
}

console.log(foo.bar());

console.log((foo.bar)());

console.log((foo.bar = foo.bar)());

console.log((false || foo.bar)());

console.log((foo.bar,foo.bar)());
```

## foo.bar()
在示例1中，MemberExpression 计算的结果是foo.bar，那么是不是一个Reference呢 ？

查看规范11.2.1 Property Accessors，这里展示了一个计算的过程，什么都不管了，就看最后一步：

我们得知该表达式返回了一个Reference类型。

根据之前的内容，我们知道该值为：

```js
var  Reference = {
  base: foo,
  name: 'bar',
  strict: false
};
```
接下来按照2.1的判断流程走：

>2.1如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)

该值是Reference类型，那么IsPropertyReference(ref)的结果是什么呢？

前面我们已经铺垫了IsPropertyReference方法，如果 base value 是一个对象，结果返回true

base value 为foo，是一个对象，所以IsPropertyReference(ref)结果为true。

这个时候我们就可以确定this的值了：

```js
this = GetBase(ref);
```

GetBase 也已经铺垫，活的base value值，这个例子中就是foo，所以this就是foo，shi例1的结果就是2。

## (foo.bar)()
看示例2：

```js
console.log((foo.bar)());
```

foo.bar被（）包住，查看规范11.1.6分组表达式。

实际上（）并没有对在示例1中，MemberExpression进行计算，所以和示例1的结果一样的。

## (foo.bar = foo.bar)()
看示例3，有赋值操作符，查看规范11.13.1 简单赋值。

计算第三步：

>3.令 rval 为 GetValue(rref).

因为使用GetValue，所以返回的值不是Reference类型，

按之前的讲的判断逻辑：

>2.3如果ref不是Reference，那么this的值为undefined

this为undefined，非严格模式下，this的值为undefined的时候，其值会被隐式转换为全局对象。

## (false || foo.bar)()
看示例4，逻辑与算法，查看规范11.11

因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined

## (foo.bar,foo.bar)()
看示例5，逗号操作符，查看规范11.14

因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined

注意：以上是在非严格模式下的结果，严格模式下因为this返回undefined，所以示例3会报错。

## 补充
最后，忘记了一个最最普通的情况：

```js
function foo() {
  console.log(this)
}
foo();
```

MemberExpression 是 foo，解析标识符，查看规范 10.3.1，会返回一个 Reference 类型的值：

```js
var fooReference = {
  base: EnvironmentRecord,
  name: 'foo',
  strict: false
};
```

接下来进行判断：

>2.1如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)

因为base value 是EnvironmentRecord，并不是一个Object，还记得前面说的base value 的取值可能吗？

只可能是undefined，Object，Boolean，String，Number和environment record中的一种。

IsPropertyReference(ref) 的结果为 false，进入下个判断：

>2.2如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 ImplicitThisValue(ref)

base value 真是Environment Record，所以会调用 ImplicitThisValue(ref)

查看规范10.2.1.1.6，ImplicitThisValue 方法的介绍：该函数始终返回 undefined。

## 下一篇文章
[JavaScript深入之执行上下文]





