## 前言
在《JavaScript深入之执行上下文栈》中讲到，当JavaScript代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。

对于每个执行上下文，都有三个重要属性

  * 变量对象(Variable object，VO)
  * 作用域链(Scope chain)
  * this

今天重点讲讲 this