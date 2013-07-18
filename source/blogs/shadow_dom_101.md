Title: [译]Shadow DOM第一课
Date: 2013-05-28 06:26:44
Tags: "Shadow DOM" "Web Components"

## 译者按

去年我曾经翻译过一篇[介绍Shadow DOM的文章](http://www.toobug.net/article/what_is_shadow_dom.html)，当时觉得这是一门好遥远的技术，但仅仅在半年之后，Chrome就已经支持了Shadow DOM，到目前为止，Web Components的各个子标准也已经初见端倪，可以再深入把玩一番了。建议使用Chrome 25+访问本文章以便可以看到可以使用的实例，否则文章中的演示将只能看到静态的图片。

## 简介

> 注：本文讨论的API还没有被完全标准化，还处在不断讨论变更的阶段，所以请在项目中谨慎使用实验性的API。

[Web Components](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html)是一系列比较前卫的标准的集合，它可以：

- 使得在页面中构建挂件（widget）成为可能
- 构建的挂件很可靠，可以被重复使用
- 如果挂件的下一个版本改变了内部实现细节，并不会影响页面

这是否是说你得在使用HTML/JavaScript还是使用Web Components之间做出选择？事实上，不是的。HTML和JavaScript可以做出交互式的组件，而挂件正是这样的交互式组件。这也就是说使用HTML和JavaScript来开发挂件是件很有意义的事情。Web Components标准正是用来做这件事情的。

> 如果构建一个挂件必须要强制使用另外的技术，那么这是没有意义的。比如，使用`<canvas>`来构建一个挂件显然是件无趣的事情。它的确很可靠，因为当你改变画布中的内容时页面不会受影响，但它在可访问性、可索引性、内容创作、自适应分辨率等方面都很不友好。

如果HTML和JavaScript来构建挂件面临一个基础性的问题，那就是挂件内的DOM树并没有被从页面其它部分封装起来。缺少封装意味着页面的CSS可能会意外地被应用到挂件内部，JavaScript可能会意外地修改挂件内部，还可能引起挂件内外的ID冲突等等。

> 缺少封装带来的另一方面的问题是，当你升级了你的库，改变了挂件的DOM结构，那么你的CSS和JavaScript有可能会意外地不能正常工作。

Web Components由5个部分组成（译注：原文中是模板、Shadow DOM、自定义元素、Packaging四个部分，翻译时根据最新规范做了修改，感谢 [@一丝](http://weibo.com/jieorlin) 提醒）：

1. [模板（Templates）](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/templates/index.html)
2. [装饰器（Decorators）](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html#decorator-section)
3. [Shadow DOM](http://www.w3.org/TR/shadow-dom/)
4. [自定义元素（Custom Elements）](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/custom/index.html)
5. [Imports](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/imports/index.html)

Shadow DOM着眼于DOM树的封装问题。Web Components的这5个部分在设计时是希望它们协同工作的，但你可以自由选择使用哪一部分。本教程将讨论如何使用Shadow DOM。

Shadow DOM目前只在Chrome 25+可用，API前面有一个`webkit`前缀。

$$solo_more$$

## Hello, Shadow World

在Shadow DOM的世界中，元素可以被关联到一种新的节点类型，这种新的节点类型叫作shadow root。一个被关联到shadow root的元素叫作shadow host。shadow host的内容不会被渲染，取而代之的是shadow root的内容。

比如，如果你有这样的代码：

	<button>Hello, world!</button>
	<script>
	var host = document.querySelector('button');
	var root = host.webkitCreateShadowRoot();
	root.textContent = 'こんにちは、影の世界!';
	</script>

页面是像这样渲染的：

<button>Hello, world!</button>
<script>
var host = document.querySelector('button');
var root = host.webkitCreateShadowRoot();
root.textContent = 'こんにちは、影の世界!';
</script>

而不是这样：

<button>Hello, world!</button>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_101_2.png" /></p>
</div>

需要注意，如果使用页面上的JavaScript来取按钮的`textContent`，得到的并不是“こんにちは、影の世界!”，而是“Hello, world!”，因为shadow root下面的DOM子树被封装起来了。

这里可能违反了一个原则，就是不应该将内容放入Shadow DOM。内容必须被放到文档中，以便被屏幕阅读器、搜索引擎、浏览器插件等读取。Shadow DOM是用来处理在构建精致好用挂件过程中面临的无语义的标记的。内容部分应该留在页面上。

> 当然，对是否将内容放入Shadow DOM这件事，我们无法强制，毕竟这是web，你可以随意做你想做的事情。不要做得太过火就好。

## 将内容从表现中抽离

现在我们来看一个使用Shadow DOM将内容从表现中抽离的例子。假设我们有这样一个名牌：

<div id="nametag1" class="outer">
	<div class="boilerplate">
		Hi! My name is
	</div>
	<div class="name">
		Bob
	</div>
</div>

> 译注：这个例子效果和原文略有差异，因为我的博客样式影响到了名牌的样式，刚好可以作为DOM不封装时写组件弊端的证明，就不修正了。

下面是代码，这是你已经每天在写的代码，没有用到Shadow DOM：

	<style>
	.outer {
		border: 2px solid brown;
		border-radius: 1em;
		background: red;
		font-size: 20pt;
		width: 12em;
		height: 7em;
		text-align: center;
	}
	.boilerplate {
		color: white;
		font-family: sans-serif;
		padding: 0.5em;
	}
	.name {
		color: black;
		background: white;
		font-family: "Marker Felt", cursive;
		font-size: 45pt;
		padding-top: 0.2em;
	}
	</style>
	<div class="outer">
		<div class="boilerplate">
			Hi! My name is
		</div>
		<div class="name">
			Bob
		</div>
	</div>

因为DOM树没有封装，名牌的整个结构都是暴露在文档中的。如果页面中碰巧有其它元素使用了相同的类名来写CSS或者JavaScript，那估计我们会很难过。

我们可以避免这种难过的日子。

### 第1步，隐藏表现的细节

从语义上讲，我们可能只关心：

- 它是一个名牌
- 名字是“Bob”

首先，我们写一个与我们期望的语义最接近的结构：

	<div id="nameTag">Bob</div>

接下来，我们将所有会用到的样式和`div`元素写到`<template>`元素中：

	<div id="nameTag">Bob</div>
	<template id="nameTagTemplate">
	<style>
	.outer {
		border: 2px solid brown;

		… same as above …

	</style>
	<div class="outer">
		<div class="boilerplate">
			Hi! My name is
		</div>
		<div class="name">
			Bob
		</div>
	</div>
	</template>

到目前为止，唯一被渲染的只有“Bob”，因为我们将用于表现的DOM结构移到了`<template>`元素中，它们没有被渲染，但我们可以从JavaScript中访问到这些DOM结构。现在，我们来处理shadow root：

	<script>
	var shadow = document.querySelector('#nameTag').webkitCreateShadowRoot();
	var template = document.querySelector('#nameTagTemplate');
	shadow.appendChild(template.content);
	template.remove();
	</script>

> 模板（Templates）和Shadow DOM一样，也是一个还未完全确定的标准。目前`<template>`元素在Chrome Canary中可用。你也可以使用你熟悉的方法如`innerHTML`，`appendChild`，`getElementById`等等。因为这篇文章是讲Shadow DOM的，所以我们不会深入去讲`template`元素是如何工作的。如果你希望了解更多，可以看[HTML's New Template Tag](http://www.html5rocks.com/tutorials/webcomponents/template/)。

现在我们有了一个shadow root，名牌又重新被渲染了。如果你在名牌标签上点右键然后审查元素，你会看到我们所期望的语义化的结构：

	<div id="nameTag">Bob</div>

从这个示例，我们可以看到，使用Shadow DOM可以将名牌的具体表现细节从文档中隐藏起来，它们被封装在了Shadow DOM中。

###　第2步，从表现中抽离内容

现在我们的名牌可以将实现细节从页面中隐藏了，但其实并没有将内容和表现分享开，因为尽管内容（“Bob”）已经在页面上了，但被渲染的内容却是来自shadow root中的副本。如果我们需要修改名牌中的名字，还需要改这两个地方，甚至可能因为一些原因导致这两处的名字并不一致。

HTML元素是可以组合的——比如你可以将一个按钮放进一个表格中。在这里我们所需要的正是组合——名牌由一个红色的背景，文本“Hi!”以及名字组合而成。

你——即挂件的作者——可以通过一个叫`<content>`的新元素来定义你的挂件将如何被组合。这将在挂件的表现中创建一个插入点（insertion point），插入点会将shadow host中的内容放到插入点所在的位置。

如果我们将Shadow DOM中的结构改成这样：

	<template id="nameTagTemplate">
	<style>
		…
	</style>
	<div class="outer">
		<div class="boilerplate">
			Hi! My name is
		</div>
		<div class="name">
			<content></content>
		</div>
	</div>
	</template>

如果这个名牌被渲染，那么shadow host的内容将被投影（project）到`<content>`元素出现的地方。

现在文档的结构显得更简单了，因为名字只出现在一个地方，即文档中。如果需要修改页面中的名字，只需要这样写就可以了：

	document.querySelector('#nameTag').textContent = 'Shellie';

浏览器会自动更新渲染结果，因为名字被投影到了`<content>`元素所在的地方。

这是一个实例：

<div id="nameTag2">Bob</div>
<template id="nameTag2Template">
<style>
.outer {
  border: 2px solid brown;
  border-radius: 1em;
  background: red;
  font-size: 20pt;
  width: 12em;
  height: 7em;
  text-align: center;
}
.boilerplate {
  color: white;
  font-family: sans-serif;
  padding: 0.5em;
}
.name {
  color: black;
  background: white;
  font-family: "Marker Felt", cursive;
  font-size: 45pt;
  padding-top: 0.2em;
  height: 55pt;
  overflow: hidden;
}
</style>
<div class="outer">
	<div class="boilerplate">
		Hi! My name is
	</div>
	<div class="name">
		<content></content>
	</div>
</div>
</template>

<p>
<label for="name2newName">New name:</label>
<input name="name2newName" value="Shellie">
<button onclick="document.querySelector('#nameTag2').innerText=document.querySelector('input[name=name2newName]').value;">Update</button>
</p>
<script>
var shadow = document.querySelector('#nameTag2').webkitCreateShadowRoot();
var template = document.querySelector('#nameTag2Template');
shadow.appendChild(template.content);
template.remove();
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_101_3.png" /></p>
</div>

现在我们完成了内容与表现的分离。内容在文档中，而表现在Shadow DOM中。如果浏览器需要渲染的话，会自动将它们进行同步。

### 第3步，好处

通过分离内容和表现，我们可以简化对内容的操作，比如在上例中，你只需要操作一个`<div>`而不是一堆DOM。

现在如果我们要修改表现，根本不需要改任何代码。

比如，如果我们要对名牌进行本地化。这个名牌在文档中的语义化结构并没有改变：

	<div id="nameTag">Bob</div>

初始化shadow root的代码仍然和上面的相同，变化的只是放入shadow root中的内容：

	<template id="nameTagTemplate">
	<style>
	.outer {
		border: 2px solid pink;
		border-radius: 1em;
		background: url(sakura.jpg);
		font-size: 20pt;
		width: 12em;
		height: 7em;
		text-align: center;
		font-family: sans-serif;
		font-weight: bold;
	}
	.name {
		font-size: 45pt;
		font-weight: normal;
		margin-top: 0.8em;
		padding-top: 0.2em;
	}
	</style>
	<div class="outer">
		<div class="name">
			<content></content>
		</div>
		と申します。
	</div>
	</template>

现在我们看到了一个日本名牌（译注：原例如此，钓鱼岛是中国的！）：

<div id="nameTag3">Bob</div>
<template id="nameTag3Template">
<style>
.outer {
	border: 2px solid pink;
	border-radius: 1em;
	background: url(/images/shadow_dom_101_1.jpg);
	font-size: 20pt;
	width: 12em;
	height: 7em;
	text-align: center;
	font-family: sans-serif;
	font-weight: bold;
}
.name {
	font-size: 45pt;
	font-weight: normal;
	margin-top: 0.8em;
	padding-top: 0.2em;
}
</style>
<div class="outer">
	<div class="name">
		<content></content>
	</div>
	と申します。
</div>
</template>

<p>
<label for="name3newName">New name:</label>
<input name="name3newName" value="Shellie">
<button onclick="document.querySelector('#nameTag3').innerText=document.querySelector('input[name=name3newName]').value;">Update</button>
</p>
<script>
var shadow = document.querySelector('#nameTag3').webkitCreateShadowRoot();
var template = document.querySelector('#nameTag3Template');
shadow.appendChild(template.content);
template.remove();
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_101_4.png" /></p>
</div>

> [背景图来自Mike Dowman](http://www.flickr.com/photos/mikedowman/5621169045/)，在Creative Commons授权下使用。

这相对于现在的web开发状况来说，是一个巨大的进步，因为用于更新名字的代码只需要依赖简单而且一致的组件结构即可，这些代码不需要知道用来做渲染的结构。具体到渲染细节上，名字出现在“Hi! My name is”之后，出现在日语之前，但对于更新名字的代码来说，这些细节都是毫无语义的，因此这些代码完全没有必要知道这些细节。

## 高级投影

在上面的例子中，`<content>`元素所在的位置放入了所有来自shadow host的元素（副本）。如果使用`select`属性，则可以选择性地进行投影。你甚至可以使用多个`content`元素。

比如，如果你的文档中结构是这样：

	<div id="nameTag">
		<div class="first">Bob</div>
		<div>B. Love</div>
		<div class="email">bob@</div>
	</div>

还有一个shadow root，它使用CSS选择器来指定内容：

	<div style="background: purple; padding: 1em;">
		<div style="color: red;">
			<content select=".first"></content>
		</div>
		<div style="color: yellow;">
			<content select="div"></content>
		</div>
		<div style="color: blue;">
			<content select=".email"></content>
		</div>
	</div>

> 注意：`select`属性只可以选择shadow host节点的近亲节点（immediate children），也就是说，你不能选择后代（如`select="table tr"`）。

`<div class="email">`同时被`<content select="div">`和`<content select=".email">`元素匹配。那么Bob的email地址会出现几次呢？是什么颜色呢？

答案是：Bob的email地址只会出现一次，是黄色的。

<div id="nameTag4">
	<div class="first">Bob</div>
	<div>B. Love</div>
	<div class="email">bob@</div>
</div>

<template id="nameTag4Template">
<div style="background: purple; padding: 1em;">
	<div style="color: red;">
		<content select=".first"></content>
	</div>
	<div style="color: yellow;">
		<content select="div"></content>
	</div>
	<div style="color: blue;">
		<content select=".email"></content>
	</div>
</div>
</template>
<script>
var shadow = document.querySelector('#nameTag4').webkitCreateShadowRoot();
var template = document.querySelector('#nameTag4Template');
shadow.appendChild(template.content);
template.remove();
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_101_5.png" /></p>
</div>

原因是，构建在屏幕上渲染的DOM树就像是一场盛大的聚会。`content`元素是邀请的发起者，它们希望文档中的内容可以进入到Shadow DOM中进行渲染的聚会。这些邀请是按顺序送达的，谁能收到邀请取决于地址上写了谁（即`select`属性）。对于内容来说，一旦接到邀请，就会立刻接受这个邀请，欣然赴宴。如果下一个邀请被送到了同样的地址，这时候，这个地址已经没有人了，所以它不会出现在聚会中。

在上面的例子中，`<div class="email">`同时匹配了`div`选择器和`.email`选择器，但因为`div`选择器来得更早一些，所以`<div class="email">`去了黄色的聚会，而没有人去蓝色的聚会。

如果有部分内容没有收到任何邀请，它们将不会被渲染。第一个例子中的“Hello, world”就是这样的情况。这种情况在你想要做一些渐进增强的渲染时很有用：在文档中写入语义化的模型，它可以被脚本获取，然后在渲染时将它隐藏，使用Shadow DOM中的渲染模型来代替它。

比如，HTML中有一个很好用的日历选择控件，如果你用`<input type="date">`就能看到一个很不错的弹出日历框。但如果你想让用户选择一个日期范围会是什么情况？你在文档中写下这样的结构：

	<div class="dateRangePicker">
		<label for="start">Start:</label>
		<input type="date" name="startDate" id="start">
		<br>
		<label for="end">End:</label>
		<input type="date" name="endDate" id="end">
	</div>

然后使用Shadow DOM创建了一个日历表格（译注：不是指datepicker中的日历，而是类似Google日历一样的表格），这个日历表格会高亮选中的日期范围。当用户点击日历表格中的日期时，组件会更新startDate输入框和endDate输入框的值，当用户提交表单时，这两个输入框中的值会被提交。

那么，为什么在文档中还要包含`label`呢，它们根本不会被渲染？原因就是，如果用户在使用一个不支持Shadow DOM的浏览器，这个表单仍然是可用的，只是没那么完美。用户将会看到类似下面这样的表单：

<div class="dateRangePicker">
	<label for="start">Start:</label>
	<input type="date" name="startDate" id="start">
	<br>
	<label for="end">End:</label>
	<input type="date" name="endDate" id="end">
</div>

## 恭喜完成Shadow DOM入门

这些就是Shadow DOM最基础的内容，恭喜你已经入门了！你可以使用Shadow DOM做更多事情，比如你可以在一个shadow host上挂多个Shadow DOM子树，或者在封装过程中将Shadow DOM嵌套起来，或者使用MDV（Model-Driven Views）和Shadow DOM来构造你的页面。事实上，Web组件远不止Shadow DOM这一门技术，比如，如果使用Web组件规范中的自定义元素部分，你可以使用声明式的方式来初始化Shadow DOM，而不必使用脚本。

我们将在稍后的课程中讲述这些内容。现在，[来Google+加入我们吧](https://plus.google.com/103330502635338602217/posts)。

> 原文地址<http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/?redirect_from_locale=zh>

<script>
	var css = document.createElement('link');
	css.setAttribute('rel','stylesheet');
	css.setAttribute('href','/attachments/shadow_dom_101_style.css');
	document.head.appendChild(css);

	if(!window.WebKitShadowRoot){
		$('.helperimg').css({

			border:'1px solid #ccc',
			background:'#eee',
			padding:'20px'

		}).show();
	}
</script>
