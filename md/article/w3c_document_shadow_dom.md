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

