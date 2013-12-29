Title: 学习ES6生成器（Generator）
Date: 2013-12-29 13:35:00
Tags: JavaScript ES6 Generator 生成器 回调

这几天，TJ大神的koa框架突然在国内火起来了，随之而来的，则是其使用的ES6生成器（Generator）引起了广大码农的强烈兴趣，各种文章也如寸后春笋般拔地而起，比如[这篇](https://www.imququ.com/post/generator-function-in-es6.html)、[这篇](http://bg.biedalian.com/2013/12/21/harmony-generator.html)、还有[这篇](https://developer.mozilla.org/zh-CN/docs/JavaScript/Guide/Iterators_and_Generators)。这个神奇的生成器被视为解决JS“回调恶魔金字塔”的利器。在动手实践之后，发现介绍ES6生成器的文章仍然有些疏漏，因此有了这篇文章，权当是对各位大大们的补充好了。

## 背景

在JS的使用场景中，异步操作的处理是一个不可回避的问题，如果不做任何抽象、组织，只是“跟着感觉走”，那么面对“按顺序发起3个ajax请求”的需求，很容易就能写出如下代码（假设已引入jQuery）：

	// 第1个ajax请求
	$.ajax({
		url:'http://echo.113.im',
		dateType:'json',
		type:'get',
		data:{
			data:JSON.stringify({status:1,data:'hello world'}),
			type:'json',
			timeout:1000
		},
		success:function(data){
			if(data.status === 1){
				// 第2个ajax请求
				$.ajax({
					......此处省略500字
					success:function(data){
						if(data.status === 1){
							// 第3个ajax请求
							$.ajax({
								......此处省略500字
								success:function(data){
									if(data.status === 1){
										
									}
								}
							});
						}
					}
				});
			}
		}
	});

当顺序执行的异步操作越来越多的时候，回调层级也就越多，这也就是传说中的“回调恶魔金字塔”，在本文章中，我们给它另一个名字“被动异步”。

还有一种场景，比如老赵的Wind.js总喜欢用的经典例子，对一个数组进行排序，但要动态展示排序过程（[详情](http://windjs.org/cn/docs/async/samples/browser/sorting-animations/)）。

在这种情况下，为了使动画能够正确呈现，我们不得不对“排序”这一本来不涉及到异步操作的逻辑做些改动，强制添加延时操作，使它变成一个异步操作。（如不理解，请搜索“JavaScript动画原理”。）

在本文中，我们给这种场景也起一个名字，叫“主动异步”。

<!-- $$solo_more$$ -->

## 生成器的卢山真面目

所谓“生成器”，其实是一个函数，但是这个函数的行为会比较特殊：

1. 它并不直接执行逻辑，而是用来生成另一个函数（这也正是“生成器”的含义）
2. 它所生成的函数可以把逻辑拆开来，一片一片调用执行，而不是像普通的函数，只能从头到尾一次执行完毕

生成器的语法和普通函数类似，特殊之处在于：

1. 字面量（函数声明/函数表达式）的关键字`function`后面多了一个`*`，而且这个`*`前后允许有空白字符
2. 函数体中多了`yield`运算符

举个粟子：

	function * GenA(){
		console.log('from GenA, first.');
		yield 1;
		console.log('from GenA, second.');
		var value3 = yield 2;
		console.log('from GenA, third.',value3);
		return 3;
	}

	var a = new GenA();

接下来依次执行：

	a.next();
	// from GenA, first.
	// Object {value:1,done:false}

	a.next();
	// from GenA, second.
	// Object {value:2,done:false}

	a.next(333);
	// from GenA, third.
	// 333
	// Object {value:3,done:true}

	a.next();
	// Error: Generator has already finished

这个例子反映了生成器的基本用法，有以下几点值得注意：

1. 在调用`GenA()`时，函数体中的逻辑并不会执行（控制台没有输出），直接调用a.next()时才会执行
2. `a`是一个对象，它由生成器`GenA()`实例化而来（事实上，不需要`new`运算符也是一样的结果）
3. 调用`a.next()`时，函数体中的逻辑才开始真正执行，每次调用时会到`yield`语句结束，并将`yield`的运算数作为结果返回
4. `a.next()`返回的结果是一个对象，对`yield`的运算数做了包装，并带上了`done`属性
5. 当`done`属性为`false`时，表示该函数逻辑还未执行完，可以调用`a.next()`继续执行，否则不可继续调用
6. 最后一次返回的结果为`return`语句返回的结果，且`done`值为`true`。如果不写`return`，则值为undefined
7. `value3 = yield 2`这句是指，这一段逻辑返回2，在下一次调用`a.next()`时，将参数赋给value3。换句话说，这句只执行了后面半段就暂停了，等到再次调用`a.next()`时才会将参数赋给value3并继续执行下面的逻辑

## 同步场景下生成器的使用

来看看同步场景下，如何使用生成器：

	function * Square(){
		for(var i=1;;i++){
			yield i*i;
		}
	}

	var square = new Square();

	square.next(); // 1
	square.next(); // 4
	square.next(); // 9
	......

同步场景下大概就是这么用的，很无趣是吧？我也这么觉得，其实和直接函数调用差别不大。不过值得注意的是，我们在循环中并没有设中止条件，因为调用一个`square.next()`方法，它才会执行一次，不调用则不执行，所以不用担心死循环的问题。

## 主动异步场景下生成器的使用

如前文所说，“主动异步”这个概念是在本文中提出来的，指那些因为某些原因需要手工延时，将同步操作变成异步操作的场景。举个简单的例子，以秒为单位，依次在console中输出1、2、3……的平方值：

	function doSquare(number){
		console.log(number * number);
		setTimeout(function(){
			doSquare(number+1);
		},1000);
	}

	doSquare(1);

如果换用生成器来做，可以这么写：

	function * Square(){
		for(var i=1;;i++){
			yield i*i;
		}
	}

	var square = new Square();
	console.log(square.next().value);
	setInterval(function(){
		console.log(square.next().value);
	},1000);

是不是觉得和同步的场景很像呀？其实就生成器`Square()`来讲，几乎是一样的，只是在调用的时候加了一个延时。这是因为生成器的特性中，并不包含异步的支持（唯一有点关联的就是上面提到的`var varible = yield value`了），所以异步的操作仍然要在其它地方处理。就这个具体的例子而言，生成器并未为我们带来任何惊喜。

未完待续……


