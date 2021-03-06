# 《JavaScript Patterns》读书笔记 第一章 介绍
_2011-12-17 17:29_

《JavaScript Patterns》(<http://book.douban.com/subject/5252901/>)是一本关于JavaScript的设计模式的书，它抛弃了其他同类书籍“一定要完全模拟面向对象”的思路，而是从一个更高的层次，即设计模式解决了什么问题入手，对每种模式进行细致的分析，然后从JavaScript本身的特性出发去进行实现，写得非常不错。

这本书目前还没有中文版，据说淘宝UED的大牛们正在翻译。

以下为读书笔记，中文为本人根据意思大致整理，不保证完全跟原文一样准确。

$$solo_more$$

### 第一章 介绍

广义上的“模式”指“重复的事件或对象的主题……它可以是一种通用的模板或者模型”

在软件开发中，一种模式就是一类问题的解决方案。一种模式不是指可以复制粘贴的代码，而是一种用来解决一类问题的抽象模板。

了解设计模式很重要，因为

1. 可以帮助我们使用已经被实践检验过的模式，不重复造轮子
2. 人脑的思维能力有限，当你思考一个复杂问题，并且注意力不在底层细节时，它能提供一种抽象的模式帮助你
3. 在不同的开发者和团队之间易于交流（注：传说中的“黑话”）

这本书讨论了设计模式、编码模式、不好的代码实现（Antipatterns）

### 一、面向对象

JavaScript是一门面向对象语言（注：似乎现在大家都认为它是“基于对象”而非“面向对象”）。

只有五种基本类型不是对象，分别是number,string,boolean,null,undefined
number,string,boolean的值很容易转变成对象，不管是被编程者还是隐式地被JavaScript解释器。

函数也是对象，可以有属性和方法。

定义变量时就在处理对象。首先，变量自动变成一个叫活动对象（或者全局对象）的属性。其次，这个变量实际上很像对象，因为它有自己的属性，决定它是否能被改变，删除或者用for-in遍历。这些属性在ECMAScript3中不是直接暴露的，但是在第5版中提供特别的装饰器手工操作它们。

对象是名值对的集合（和其它语言的关联数组的概念很像）。有的属性值可以是函数，这些函数被叫作方法。

### 二、没有类

在JS中创建对象不需要类，只需要创建它，然后给它添加基本类型、函数或者对象作为属性即可。一个空对象不是真的是空的，它有一些原生的属性，但是它没有自己的（own）属性。

### 三、原型

JavaScript有继承，虽然这只是一种代码复用的方式。继承可以通过多种不同的方式完成，经常会利用到原型。

原型是一个对象，每个由编码者创建的函数都自动有一个指向空对象的原型，这个对象跟使用字面量或者Object()构造函数创建的空对象差不多，除了constructor是指向函数而不是原生的Object()

### 四、环境

JavaScript需要一个环境来运行，最常见的是浏览器，但那不是唯一的环境。

这本书的大部分是讲的JavaScript核心部分，与环境无关。

环境可以提供自己的宿主对象，这是在ECMAScript中没有定义的，可能会有意料之外的行为（注：如浏览器的JS兼容性问题就绝大部分来源于宿主对象的差异）。

### 五、ECMAScript5

JavaScript核心部分基于ECMAScript标准。第三版标准于1999年被官方承认，也是现在各浏览器实现的版本。第四版已经放弃。第五版在2009年发布。

第5版加入了一些原生对象、方法和属性。但是最重要的变化是严格模式（strict mode），这个模式从以前的版本中移除了一些特性，使得编程更简单，减少错误倾向。

严格模式通过一段普通的文本触发，向下兼容。

在一个作用域中（不管是函数作用域、全局作用域，还是传给eval()的字符串的开头），都可以用”user strict”来触发严格模式。

这本书不研究与ES5有关的模式，因为成书时没有浏览器实现了ES5。但是例子中与ES5相关的特性如下：

1. 保证提供的代码在严格模式下不报错
2. 避免使用ES5弃用的结构，如arguments.callee
3. 尽量使用ES5中原生的方法（注：在ES3中自己实现，保持名称和参数一样），如Object.create()

> 注：后面的JSLint和console，略

> 2012年8月20日注：目前我正在参与翻译该书，[翻译稿](https://github.com/TooooBug/javascript.patterns)可以在GitHub上找到。