Title: [译]Shadow DOM第三课
Date: 2013-08-10 11:24:45
Tags: "Shadow DOM" "Web Components"

本文将会讨论Shadow DOM的更多神奇之处。本文是在[《Shadow DOM第一课》](http://www.toobug.net/article/shadow_dom_101.html)和[《Shadow DOM第二课》](http://www.toobug.net/article/shadow_dom_201.html)的基础之上讨论的，如果你需要基础介绍，请参看那两篇。

## 使用多个shadow root

如果你要开个party，让所有人都呆在同一个房间，很快就会让这个房子变得很闷，你可能更希望将不同的人群分配到不同的房间。用于挂载shadow dom的元素（shadow host）也可以这样做，也就是说，它们可以同时挂载多个shadow root。

<!-- $$solo_more$$ -->

我们来看看，往一个shadow host上挂载多个shadow root时会发生什么：

	<div id="example1">Host node</div>
	<script>
	var container = document.querySelector('#example1');
	var root1 = container.webkitCreateShadowRoot();
	var root2 = container.webkitCreateShadowRoot();
	root1.innerHTML = '<div>Root 1 FTW</div>';
	root2.innerHTML = '<div>Root 2 FTW</div>';
	</script>

在开发者工具中这样显示：

![开发者工具中显示shadow host挂载多个shadow root](/images/shadow_dom_301_1.png)

> 注意：需要在开发者工具中打开“Show Shadow DOM”选项才可以看到shadow root。

实例：

<div id="example1">Host node</div>
<script>
var container = document.querySelector('#example1');
var root1 = container.webkitCreateShadowRoot();
var root2 = container.webkitCreateShadowRoot();
root1.innerHTML = '<div>Root 1 FTW</div>';
root2.innerHTML = '<div>Root 2 FTW</div>';
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_301_2.png" /></p>
</div>

尽管我们确实挂了两个shadow root，但真正被渲染的是“Root 2 FTW”。这是因为挂载在同一个shadow host上的多个shadow root，最后一个会被渲染。在渲染的时候是采用“后进先出”（LIFO，即Last In First Out）的原则。从开发者工具中看到的结果也可以验证这种行为。

> 在shadow host上添加的shadow root（shadow子树）会以被添加的顺序放到一个栈（stack）中，最上面是最后添加的元素。所以最后添加的那个就是真正被渲染的元素。

> 最后被添加的子树叫作younger tree，而最新添加的树叫作older tree。在这个例子中，`root2`就是younger truee，而`tree1`就是older tree。

既然只有最后一个子树会被渲染，那如何使用多个shadow子树呢？答案就是shadow插入点（shadow insertion point）。

### shadow插入点

shadow插入点（`<shadow>`）和普通的插入点（`<content>`）很相似，都是“占位符”。但是，shadow插入点并不是用来插入shadow host的内容，而是用来挂载其它的shadow子树，也就是其它shadow子树开始的地方。

> 如果在shadow子树中有多个`<shadow>`元素，只有第一个会被识别，其它的会被忽略。

回头看我们的例子，第一个shadow子树`root1`并没有被加入到渲染列表。添加一个`<shadow>`元素的话就可以让它也渲染。

	<div id="example2">Host node</div>
	<script>
	var container = document.querySelector('#example2');
	var root1 = container.webkitCreateShadowRoot();
	var root2 = container.webkitCreateShadowRoot();
	root1.innerHTML = '<div>Root 1 FTW</div><content></content>';
	root2.innerHTML = '<div>Root 2 FTW</div><shadow></shadow>';
	</script>

在开发者工具中这样显示：

![使用shadow插入点挂载多个shadow root](/images/shadow_dom_301_3.png)

> 注意：需要在开发者工具中打开“Show Shadow DOM”选项才可以看到shadow root。

实例：

<div id="example2">Host node</div>
<script>
var container = document.querySelector('#example2');
var root1 = container.webkitCreateShadowRoot();
var root2 = container.webkitCreateShadowRoot();
root1.innerHTML = '<div>Root 1 FTW</div><content></content>';
root2.innerHTML = '<div>Root 2 FTW</div><shadow></shadow>';
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_301_4.png" /></p>
</div>

在这个例子中有几点值得注意的地方：

1. “Root 2 FTW”仍然渲染在“Root 1 FTW”上方。这是因为，我们放的shadow插入点在内容之后。如果你想反过来，可以把shadow插入点放到前面：

		root2.innerHTML = '<shadow></shadow><div>Root 2 FTW</div>';

2. 注意现在在root1中有了一个`<content>`插入点。这使得shadow host的文本“Host node”也可以参与渲染。

### 在`<shadow>`所在的地方渲染的是谁？

有时候，会很想知道在`<shadow>`元素处渲染的是哪个子树。你可以通过`.olderShadowRoot`来引用：

	root2.querySelector('shadow').olderShadowRoot === root1 //true

`.olderShadowRoot`没有带上浏览器产商前缀，因为`HTMLShadowElement`只在Shadow DOM中才有意义，而Shadow DOM已经加了前缀了。

## 观察shadow host上挂载的shadow root

如果一个元素上挂载了Shadow DOM，那么你可以通过`.webkitShadowRoot`来访问youngest shadow root（最后挂载的shadow root）：

	var root = host.webkitCreateShadowRoot();
	console.log(host.webkitShadowRoot === root); // true
	console.log(document.body.webkitShadowRoot); // null

> 我不是很确定为什么要提供`.shadowRoot`。因为它其实破坏了Shadow DOM的封装，给了外界访问本来应该隐藏的DOM的机会。

如果你担心外界侵入到你的Shadow DOM，你可以重写`.shadowRoot`：

	Object.defineProperty(host, 'webkitShadowRoot', {
		get: function() { return null; },
		set: function(value) { }
	});

有点取巧，但确定有效。这样做也是为了寻找一种保持Shadow DOM私有性的方法。

最后还有一点，在玩Shadow DOM时应该意识到，Shadow DOM并不是那么安全。不应该依赖它来让组件保持完全独立。

## 在JS中创建Shadow DOM

如果你喜欢在JS中创建DOM元素，那么可以使用`HTMLContentElement`和`HTMLShadowElement`。

	<div id="example3">
		<span>Host node</span>
	</div>
	<script>
	var container = document.querySelector('#example3');
	var root1 = container.webkitCreateShadowRoot();
	var root2 = container.webkitCreateShadowRoot();

	var div = document.createElement('div');
	div.textContent = 'Root 1 FTW';
	root1.appendChild(div);

	// HTMLContentElement
	var content = document.createElement('content');
	content.select = 'span'; // selects any spans the host node contains
	root1.appendChild(content);

	var div = document.createElement('div');
	div.textContent = 'Root 2 FTW';
	root2.appendChild(div);

	// HTMLShadowElement
	var shadow = document.createElement('shadow');
	root2.appendChild(shadow);
	</script>

这个例子和前面那个很像。唯一的区别是现在用了一个`select`来选择新增的`<span>`元素。

## 获取被分配的节点

被`select`选中的shadow host的子节点会被“分配”到shadow子树中，它们叫……当当当……被分配的元素！当有插入点选择它们时，就会穿越shadow边界，加入渲染名单。

在概念上，有一个比较古怪的地方，就是插入点并没有真正移动DOM元素。shadow host的子元素仍然安静地呆在原来的地方，插入点只是将它们“投影”到了shadow子树中。这仅仅是一件渲染时发生的事情：<s>“把这些元素移过来”</s>“在这个位置渲染这此元素”。

注意：你无法把一个DOM放到`<content>`中。如：

	<div><h2>Host node</h2></div>
	<script>
	var shadowRoot = document.querySelector('div').webkitCreateShadowRoot();
	shadowRoot.innerHTML = '<content select="h2"></content>';

	var h2 = document.querySelector('h2');
	console.log(shadowRoot.querySelector('content[select="h2"] h2')); // null;
	console.log(shadowRoot.querySelector('content').contains(h2)); // false
	</script>

上面的`h2`并不是Shadow DOM的子元素。这导致了一个有趣的东西：

> 插入点强大得出乎意料，你可以把它想象成一种创建Shadow DOM“声明式API”（declarative API）的方式。shadow host可以包含全世界所有的标记，但如果Shadow DOM不通过插入点使用它们，它们就永远没意义。

### Element.getDistributedNodes()

我们不能直接访问到`<content>`里面的内容，但是有一个API `.getDistributedNodes()`可以用来选择被分配到插入点的元素：

	<div id="example4">
		<h2>Eric</h2>
		<h2>Bidelman</h2>
		<div>Digital Jedi</div>
		<h4>footer text</h4>
	</div>

	<template id="sdom">
		<header>
			<content select="h2"></content>
		</header>
		<section>
			<content select="div"></content>
		</section>
		<footer>
			<content select="h4:first-of-type"></content>
		</footer>
	</template>

	<script>
	var container = document.querySelector('#example4');

	var root = container.webkitCreateShadowRoot();
	root.appendChild(document.querySelector('#sdom').content.cloneNode(true));

	var html = [];
	[].forEach.call(root.querySelectorAll('content'), function(el) {
		html.push(el.outerHTML + ': ');
		var nodes = el.getDistributedNodes();
		[].forEach.call(nodes, function(node) {
			html.push(node.outerHTML);
		});
		html.push('\n');
	});
	</script>

输出：

	<content select="h2"></content>: <h2>Eric</h2><h2>Bidelman</h2>
	<content select="div"></content>: <div>Digital Jedi</div>
	<content select="h4:first-of-type"></content>: <h4>footer text</h4>

## 工具：Shadow DOM可视化

理解Shadown DOM还是多少有些困难的。当我刚开始接触的时候花了不少时间去学习。

为了能让Shadow DOM的渲染过程能更直接，我使用[d3.js](http://d3js.org/)创建了一个可视化工具。在左边显示的两块代码都是可以编辑的。你可以粘贴自己的代码进去，然后看看元素是怎么进入的插入点的，是怎么渲染的。

![Shadow DOM可视化工具](/images/shadow_dom_301_5.png)

你可以[点击这里](http://html5-demos.appspot.com/static/shadowdom-visualizer/index.html)进入这个可视化工具，一定要试一下，然后告诉我你的意见哈。

<embed src="http://player.youku.com/player.php/sid/XNTk2Njc3MDk2/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

## 事件模型

有一些事件会穿越shadow边界，有一些不会。在事件穿越边界的情况下，事件目标（event target）将会被调整，以保持shadow root上边界的封装性。具体地说，调整后的事件看起来是来自shadow host，而不是Shadow DOM内部。

如果你的浏览器支持Shadow DOM，你应该可以看到下面的操作区，用它可以直观地反映事件的情况。黄色元素是Shadow DOM中的标记，蓝色元素是shadow host的一部分，包裹在写有“I'm a node in the host”元素外面的是黄色边框，表示它是一个被分配到`<content>`插入点中的元素。

“Play Action”按钮会展示中不同的试验方法（鼠标如何操作）。你可以试一下，然后看看`mouseout`和`focusin`事件分别是如何冒泡的。


<div id="example5" class="demoarea">
	<div data-host>
		<div class="blue">I'm a node in the host</div>
	</div>

	<template style="display:none;"> <!-- display:none used for older browsers -->
		<style>
		.scopestyleforolderbrowsers * {
			border: 4px solid #FC0;
		}
		.scopestyleforolderbrowsers input {
			padding: 5px;
		}
		.scopestyleforolderbrowsers div {
			background: #FC0;
			padding: 5px;
			border-radius: 3px;
			margin: 5px 0;
		}
		content::-webkit-distributed(*) {
			border: 4px solid #FC0;
		}
		</style>
		<section class="scopestyleforolderbrowsers">
			<div>I'm a node in Shadow DOM</div>
			<div>I'm a node in Shadow DOM</div>
			<content></content>
			<input type="text" placeholder="I'm in Shadow DOM">
			<div>I'm a node in Shadow DOM</div>
			<div>I'm a node in Shadow DOM</div>
		</section>
	</template>

	<aside class="cursor"></aside>

	<div class="buttons">
		<button data-action="playAnimation" data-action-idx="1">Play Action 1</button><br>
		<button data-action="playAnimation" data-action-idx="2">Play Action 2</button><br>
		<button data-action="playAnimation" data-action-idx="3">Play Action 3</button><br>
		<button data-action="clearLog">Clear log</button>
	</div>

	<output></output>
</div>

<script>
(function() {
function stringify(node) {
	return node.outerHTML.match(".*?>")[0].replace('<', '&lt;').replace('>', '&gt;');
}

var out = document.querySelector('#example5 output');
var host = document.querySelector('#example5 [data-host]');
var wrapper = document.querySelector('#example5');

var root = host.webkitCreateShadowRoot();
root.innerHTML = document.querySelector('#example5 template').innerHTML;

host.addEventListener('mouseout', function(e) {
	out.innerHTML += [
		'<span>[' + e.type + ']</span>', 
		'on:', stringify(e.target) + ',', 
		'from', stringify(e.fromElement),
		'&rarr;', stringify(e.toElement), '<br>'].join(' ');
	out.scrollTop = out.scrollHeight;
});

document.addEventListener('focusin', function(e) {
	out.innerHTML += [
		'<span>[' + e.type + ']</span>',
		'on:', stringify(e.target), '<br>'].join(' ');
	out.scrollTop = out.scrollHeight;
});

function clearLog() {
	out.innerHTML = '';
}

function cleanUpAnimations(node) {
	[].forEach.call(node.classList, function(c) {
		if (c.indexOf('animation') == 0) {
			node.classList.remove(c);
		}
	});
}

function playAnimation(idx) {
	clearLog();
	wrapper.classList.add('playing');
	wrapper.classList.add('animation' + idx);
}

wrapper.addEventListener('webkitAnimationEnd', function(e) {
	this.classList.remove('playing');
	cleanUpAnimations(this);
});

document.querySelector('#example5 .buttons').addEventListener('click', function(e) {
	if (e.target.tagName == 'BUTTON') {
		switch(e.target.dataset.action) {
			case 'clearLog':
				clearLog();
				break;
			case 'playAnimation':
				cleanUpAnimations(wrapper);
				playAnimation(parseInt(e.target.dataset.actionIdx));
				break;
			default:
				break;
		}
	}
});

})();
</script>


<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_301_6.png" /></p>
</div>

- Play Action1

	这个很有意思。你应该可以看到`mouseout`事件从shadow host元素（`<div data-host>`）冒泡到`<div class="blue">`元素。尽管它是一个被分配的元素，但是它仍然存在于shadow host中，而不是Shadow DOM中。接着移动鼠标，会看到又一个由`<div class="blue">`元素触发的`mouseout`事件冒泡到shadow host中。（译注：这里shadow host和`.blue`是父子关系，从父元素进入子元素会触发父元素的`mouseout`，从子元素移回父元素则有子元素的`mouseout`。）

- Play Action2

	会有一个`mouseout`事件发生在shadow host元素上（移动到最下方时）。正常来说，你应该看到在从每个黄色块移出时都有一个`mouseout`事件。但，在这个例子中，黄色元素都在Shadow DOM内部，这些事件不会穿越它们的上边界。

- Play Action3

	注意，当你点击输入框时，`focusin`看起来不是发生在输入框上的，而是在shadow host元素上。事件目标被调整了。


### 不会冒泡的事件

下面的事件不会穿越shadow边界：

- `abort`
- `error`
- `select`
- `change`
- `load`
- `reset`
- `resize`
- `scroll`
- `selectstart`

## 总结

Shadow DOM异常强大，希望你在看完文章后同意这句话。因为它，我们第一次有了不使用`<iframe>`（或者其它古老的方式）来封装元素的方法。

必须承认，Shadow DOM很复杂，但它加入web平台后绝对是有价值的。尽管去花点时间学习吧。

如果你希望学习更多，请参见[《Shadow DOM第一课》](http://www.toobug.net/article/shadow_dom_101.html)和[《Shadow DOM第二课》](http://www.toobug.net/article/shadow_dom_201.html)。

谢谢[Dominic Cooney](http://www.html5rocks.com/profiles/#dominiccooney)和[Dimitri Glakov](https://plus.google.com/111648463906387632236/posts)帮助审查本文。

> 原文地址<http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-301/>

<script>
	var css = document.createElement('link');
	css.setAttribute('rel','stylesheet');
	css.setAttribute('href','/attachments/shadow_dom_301_style.css');
	document.head.appendChild(css);

	if(!window.WebKitShadowRoot){
		$('.helperimg').css({

			border:'1px solid #ccc',
			background:'#eee',
			padding:'20px'

		}).show();
	}
</script>
