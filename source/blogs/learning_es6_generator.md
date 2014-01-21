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

1. 它并不直接执行逻辑，而是用来生成另一个对象（这也正是“生成器”的含义）
2. 它所生成的对象中的函数可以把逻辑拆开来，一片一片调用执行，而不是像普通的函数，只能从头到尾一次执行完毕

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
6. 最后一次返回的结果为`return`语句返回的结果，且`done`值为`true`。如果不写`return`，则值为`undefined`
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

## 被动异步场景下的生成器使用

如前文所说，“被动异步”是本文中约定的概念，指那些操作本身就是异步的，没有办法将延时和操作本身分享开来的操作，比如ajax请求，没办法将请求和延时分离开处理。（上例“主动异步”中的延时则是手工加的，既可以放在`Square`中，也可以放在`Square`外。）

那么，如何用生成器解决这种被动异步场景下的“回调恶魔金字塔”呢？满心期待对吧，很遗憾，它并不能那么简单地解决……

从前面的例子中，其实已经可以体会出来了，生成器的用法中并不包含对异步的处理，所以其实没有办法帮助我们对异步回调进行封闭。那么为什么大家将它视为解决回调嵌套的神器呢？在翻阅了不少资料后找到[这篇文章](http://blog.stevensanderson.com/2013/12/21/experiments-with-koa-and-javascript-generators/)，文章作者一开始也认为生成器并不能解决回调嵌套的问题，但下面自己做了解释，如果生成器的返回的是一系列的Promise对象的话，情况就会不一样了，举个粟子：

	function myAjax1(){
		return $.getJSON('http://echo.113.im',{
			data:JSON.stringify({data:1}),
			type:'json'
		});
	}

我们使用jQuery中的getJSON方法还处理ajax请求，这个方法会返回一个Promise对象。然后，我们使用一个生成器来包装这个操作：

	function * MyLogic(){
		var serverData = yield myAjax1();
		console.log(serverData)
	}

使用的时候这样用：

	var myLogic = new MyLogic();
	var promise = myLogic.next().value;
	promise.done(function(serverData){
		myLogic.next(serverData);
	});

可以看到，我们这里的`myAjax1()`以及`MyLogic()`函数中，并没有使用回调，就完成了异步操作。你一定会问，下面这个`promise.done`不就是回调操作么？Bingo！这正是精华所在！我们来看一下这段代码做了什么：

首先，`myLogic.next()`返回了一个Promise对象（`promise`），然后，`promise.done`中的回调函数所做的事情就是调用`next`方法就行了，除了调用`next`方法，其它的什么事情都没有。此时，我们就会想到一个程序员特别喜欢的词，叫“封装”！既然这个回调函数只是调用`next`方法，那为什么不把它封装起来？

了解到这里，再去看[这篇](http://bg.biedalian.com/2013/12/21/harmony-generator.html)文章中所说的`co`函数，相信你就恍然大悟了！这个`co`函数正是在封装调用`next`方法这件事情！

	function co(GenFunc) {
		return function(cb) {
			var gen = GenFunc()
			next()
			function next() {
				if (gen.next) {
					var ret = gen.next()
					if (ret.done) { // 如果结束就执行cb
						cb && cb()
					} else { // 继续next
						ret.value(next)
					}
				}
			}
		}
	}

当我们把这个细节屏蔽之后，再回头去看我们的异步代码，是不是就没有回调了？！哇噻，怎么办到的？好神奇啊！

最后，以别人文章中的一段koa框架使用代码收尾吧：

	var koa = require('koa'),
		app = koa();
	 
	app.use(function *() {

		// 这是这个例子中最重要的部分，我们进行了一系列异步操作，却没有回调
		var city = yield geolocation.getCityAsync(this.req.ip);
		var forecast = yield weather.getForecastAsync(city);
	 
		this.body = 'Today, ' + city + ' will be ' + forecast.temperature + ' degrees.';

	});
	 
	app.listen(8080);

看到了吧，koa正是封装了对生成器返回值的处理和调用`next`方法的细节（这里的`app.use`就像前面的`co`函数），使得我们的逻辑代码看起来是如此简单，这正是koa的伟大之处，也是ES6生成器这一特性能迅速引起如此多轰动的真正原因。

> P.S: 本文中的“主动异步”和“被动异步”其实都可以用同样的方式来封装，中间加入“主动同步”的内容只是为了正好地理解异步场景。另外对Promise不了解的同学建议先了解Promise的基本用法再理解生成器比较好。
