# W3C Shadow DOM中译酱油版
_2013-01-01 15:40:15_

> 前言：Shadow DOM是一种用于封闭web组件的技术。之前我也翻译过[一篇介绍Shadow DOM的文章](http://www.toobug.net/article/what_is_shadow_dom.html)，但那篇主要是从原理上讲解。现在，W3C的文档已经相对成熟，Chrome 23已经可以支持Shadow DOM，也确有必要再进一步了解这个技术了。本文的翻译方式是边理解边翻译，因此不会逐字逐句翻译，能讲清楚意思就好。

## 摘要

本文档描述了一种用来达成和维护DOM子树的方法以及这些子树之前通讯的方式。这种方法会使得DOM的功能封装比现有方法更好。

## 本文档状态

> 本节描述了本文档被发布时的状态，其它文档可能会取代本文档。W3C已经发布的文档和本技术文档的最新版本的列表可以在[W3C技术报告列表](http://www.w3.org/TR)找到。

此处废话较多，摘要如下：本文档由[Web Applications Working Group](http://www.w3.org/2008/webapps/)作为草稿发布。草稿意味着以后可能会被修改、废弃。

## 目录

省略……

## 一、关于本文档

省略，主要是对于文档中的“MUST”、“MUST NOT”之类的词语进行约定，说明文档的结构，约束浏览器行为。

## 二、依赖

省略，主要是说明文档依赖于CSS、DOM等规范文档。

## 三、术语

省略，讲DOM、document、node、element、DOM树、DOM结构的定义。

## 四、介绍

Web应用开发者常遇到的一个问题就是封闭DOM结构。尽管是页面DOM结构的一部分，但很明显有很多比较典型的（独立）DOM片段（DOM子树），我们通常也会认为它们的操作也是独立的。这种类型的封装我们叫作“功能性封装”（functional encapsulation），与基于信任关系的用限制信息流来保证安全的“信任封装”（trust encapsulation）相对。

功能性封装主要处理文档树中的“功能边界”（functional boundary）。功能边界主要用来划定两个相对独立的功能单元。

### 1、功能性封装示例

一个Web应用的界面通常由几个部分（widget）组成，每个部分都是一棵DOM子树。假如有一个widget负责放置其它的widget，那么它就会想要知道哪里是一棵DOM子树的结束，哪里是另一棵DOM子树的开始。

TODO:图

在文档树中保持功能边界的需求在一个widget被外部操作的时候就更为强烈。比如将一个widget放入一个web应用中时，除非应用确切地知道widget的DOM结构是如何设计的，否则应用根本不可能合理地操作widget。一个常见的解决办法是由widget的开发者在文档中提供一套和DOM操作类似的API。

## 五、Shadow DOM子树

为了解决这个问题，一个新的概念被提出来了。Shadow DOM允许（文档树中的）多棵DOM子树在渲染的时候被合成为一棵更大的树。通过让文档树中的任意元素承载（或者挂接？原文host）一棵或者多棵子树，就可以达到文档树中存在多棵子树的目的。这些Shadow DOM子树被一系列的规则约束以达到封装的目的，同时也可以保留DOM本来的语义。

封装之后在Shadow DOM之间的边界被叫作“shadow边界”。用于承载shadow DOM子树的元素叫作“shadow host”。Shadow DOM子树的根元素叫作“shadow root”。

TODO:图

渲染的时候，Shadow DOM子树会替换shadow host的内容。

TODO:图

为了能够将shadow host的子元素和Shadow DOM子树组合起来，一个叫作“插入点”（insertion point）的概念被提出来。插入点是在Shadow DOM子树中定义的一个位置，在渲染的时候，shadow host的子元素被会放到插入点的位置上。用于决定shadow host的哪个子元素放到哪个插入点的机制叫作“分配”

TODO:图

这样，Shadow DOM子树的封装就可以分为两部分来看：

1. 上边界封装（upper-boundary encapsulation），即用来管理shadow root和shadow host之间的边界
2. 下边界封装（lower-boundary encapsulation），即用来管理shadow host的子元素和插入点之间的边界

### 1、上边界封装

为了维护上边界封装，以下规则必须被应用到Shadow DOM子树的所有节点上（好麻烦的表述，大意是两句话，不愿看的可以跳过：第一句，子树中的节点不可以从文档中访问；第二句，子树中的节点可以从同一子树中访问）：

- ownerDocument属性，指向shadow host所属的文档
- 节点和具名元素（译注：不严谨地说，就是含name属性的元素）不可以通过shadow host的DOM tree accessors（译注：如document.head）或者通过window对象的具名属性访问
- 节点不在文档中的任何NodeList、HTMLCollection、DOMElementMap实例中出现
- 有唯一ID的或者是具名元素不可以在shadow host所属的文档中通过任何属性定位到
- 节点的样式不可以通过shadow host所属的文档的CSSOM extensions访问到
- 节点可以通过shadow root的DOM tree accessors访问到
- 有唯一ID的或者是具名元素可以被同一Shadow DOM子树中元素的任何属性定位到
- selector不可以穿越shadow边界，从文档树影响到Shadow DOM子树中

为方便起见，shadow root提供了自己的一套DOM tree accessor方法，只有自己的子孙节点可以使用。

shadow root的parentNode和parentElement必须返回null。

### 2、下边界封装

为了维护下边界封闭，分配shadow host的子节点到Shadow DOM子树中的插入点的过程必须有以下规则：

- 分配过程不影响文档DOM和Shadow DOM子树的状态
- 分配过程中参与分配的插入点通过提供一个规则（criteria）来指定子节点，这个规则决定某个节点是否被分配到指定的插入点
- 分配过程是执行一个稳定的算法的结果（译注：应该是指不存在随机性）
- 分配过程不会改变影响分配结果的变量的值
- 当影响分配结果的变量值改变的时候，分配过程会重新进行

一个插入点可能是“活动的”（active）或者是“非活动的”（inactive）。活动的插入点会参与分配过程，非活动的则不会参与。如果没有显式设为非活动，则插入点默认是活动的。

如果一个插入点不在Shadow DOM子树中，它必须和HTMLUnknownElement一样渲染。（译注：没懂这种情况如何出现……）

分配算法必须有输出，等价于执行以下几步（这里和伪代码差不多了，可跳过）：

> 输入
> 	TREE，Shadow DOM子树
> 	POOL，DOM节点的列表
> 输出
> 	POOL中的节点被分配到TREE中的插入点
> 1. 按照顺序遍历TREE中活动的插入点：
> 	1. 让POINT成为当前的插入点
> 	2. 遍历POOL中的节点：
> 		1. 让NODE成为当前节点
> 		2. 如果NODE和POINT的规则匹配：
> 			1. 将NODE分配给POINT
> 			2. 将NODE从POOL中移除
> 		3. 否则，继续遍历
> 	3. 继续遍历

### 3、匹配插入点

插入点的匹配规则是通过一系列“选择器片段”（selector fragment）来定义的。一个选择器片段实际上是这样一个选择器中的片段(shadow-host)>(fragment)，其中(shadow-host)是一个唯一标识shadow host的选择器，(fragment)是一个选择器片段。（译注：即选择器片段特指fragment部分，不包含shadow host。）

一个有效的选择器片段可以包含：

- 节点类型选择器（如div）或者通用选择器（*）
- class选择器
- ID选择器
- 属性选择器
- 以下列出的伪类选择器：
	- :link
	- :visited
	- :target
	- :enabled
	- :disabled
	- :checked
	- :indeterminate
	- :nth-child()
	- :nth-last-child()
	- :nth-of-type()
	- :nth-last-of-type()
	- :first-child
	- :last-child
	- :first-of-type
	- :last-of-type
	- :only-of-type

如果其它类型的选择器出现在选择器片段上，这个片段必须被认为是无效的。

浏览器必须让节点去匹配选择器片段，如果节点（译注：不确定是不是这个意思，原文“A conforming UAs must consider a node as matching a set of selector fragments in the context of a given shadow host, if it:”）：

- 是shadow host的子节点，且
- 所有的选择器片段是有效的，且
- 子节点至少匹配一个选择器片段，或者选择器片段是空的

### 4、匹配子元素，分配到插入点

[“引用组合器”（Reference combinators）](http://dev.w3.org/csswg/selectors4/#idref-combinators)（译注：这是属于[选择器标准](http://dev.w3.org/csswg/selectors4)的一个概念，大意是“A /attr/ B”表示当A的attr属性值为B的ID时，选择B。其中A、B被叫作“复合选择器”，attr被叫作“组合器”。标准中有一个CSS的例子类似于“label:hover /for/ input”，即label的for属性为input的ID，且label处于hover态时应用到input上的样式）被用来匹配shadow host的子节点，并将它们分配到Shadow DOM子树中的插入点上。匹配过程中必须应用以下规则：

- 组合器的值必须是select
- 第一个复合选择器必须匹配到插入点
- 第二个复合选择器必须匹配一个要被分配到这个插入点的元素

例如，.some-insertion-point /select/ div.special将匹配所有class中含special的div，并将它们分配到class中含有some-insertion-point的插入点。

所有其它类型的选择器不能跨越Shadow DOM子树和shadow host之间的shadow边界。

### 5、承载多棵Shadow DOM子树

一个shadow host可以承载多棵Shadow DOM子树。这种情况下，子树按照被添加的顺序进入堆栈，（遍历时）从最近添加的子树开始。放置树的东东叫“树栈”（tree stack）。最近被添加的树叫“新树”（younger tree），最先添加的树叫“老树”（older tree）。最后被添加的那棵树叫“最新树”（youngest tree）。

为了方便组合同一个shadow host上面的多棵子树，一种特殊的插入点（“shadow插入点”）被提出来了。shadow插入点是在Shadow DOM子树中指定的一个位置，用来标示渲染时将“老树”插在哪里。如果一棵Shadow DOM子树中有多个shadow插入点，只有第一个是有效的。这也就是说一棵Shadow DOM子树会被分配到一个shadow插入点的位置。（译注：指除“最新树”以外的，因为最新树是直接插在shadow root上的。）

像其它的插入点一样，shadow插入点也可以是活动的和非活动的。

组合是通过“树组合算法”完成，必须等价于以下步骤（又来伪代码了，跳吧……）：

> 输入
> 	HOST，一个shadow host
> 输出
> 	所有的插入点和shadow插入点被填充
> 1. 让TREE成为HOST中的“树栈”中的“最新树”
> 2. 让POOL成为节点列表
> 3. 将HOST的子元素填充进POOL中
> 4. 当TREE存在时：
> 	1. 让POINT成为TREE中第一个活动的shadow插入点
> 	2. 运行“分配算法”（译注：上文中的），提供POOL和TREE作为输入
> 	3. 如果POINT存在：
> 		1. 寻找在HOST中比TREE老一点的树
> 			1. 如果没有更老的树：
> 				1. 将POOL中的节点分配给POINT
> 				2. 停止
> 			2. 否则：
> 				1. 让TREE成为这棵老一点的树
> 				2. 将TREE放置到POINT
> 				3. 继续循环
> 	4. 否则，停止

当一个插入点或者shadow插入点没有被放置或者分配到任何东西时，在渲染时必须用回退（降级）内容替代它们。回退内容是指代表插入点的DOM元素的所有子元素。在回退内容中出现的插入点或者shadow插入点必须被认为是非活动的。

### 6、嵌套的Shadow DOM子树

Shadow DOM子树中的任何元素都可以是shadow host，这样就可以产生嵌套的Shadow DOM子树。当一棵Shadow DOM子树对应的shadow host本身是另一棵Shadow DOM子树的一部分时，这棵子树将被嵌套。

当一棵Shadow DOM子树被另一棵通过一级或者更多级嵌套时，称这棵子树被另一棵子树封闭（enclosed）。

### 7、渲染Shadow DOM子树

渲染Shadow DOM子树，或者叫将它们可视化的过程，被专门定义，必须按照以下步骤进行（伪代码又来了，不爱译了，闪人……原文在[这里](http://www.w3.org/TR/shadow-dom/#rendering-shadow-subtrees)）：

(伪代码，省略……)

渲染的结果会产生多棵DOM子树组合的一个新结构，其中包含文档树。我们用“渲染后的”来指代这个结构。



