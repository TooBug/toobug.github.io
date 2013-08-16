# HTML续行符
_2012-02-01 18:04_

常识：

1. HTML源码中夹在文字中连续的空白（空格、回车、TAB等）会在页面上形成一个空格
2. 大部分的代码规范会规定单行不得超过XX字符，超过必须做折行处理
3. CSS可以按属性多行编写，JS的长字符串可以用“\”续行

问题：HTML的长字符串怎么办？

答案：使用注释

例：

	这是源码中的第一行文字，它很长，以至于达到了要折行的要求<!--
	-->接着写第二，不管它有多长都不怕了，因为在页面中，始终是<!--
	-->当成同一行处理。

> 2012年8月17日注：这种用法除了是因为规范外，还有一个原因，就是消除inline-block带来的间隙，详见[inline-block的前世今生](http://ued.taobao.com/blog/2012/08/15/inline-block/)。