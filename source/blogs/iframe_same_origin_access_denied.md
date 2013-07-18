# 同域iframe“拒绝访问”的问题
_2011-12-01 16:57_

同事项目中用了一个iframe，父页和子页都设置了document.domain进行降域，结果在非IE下正常，IE下“拒绝访问”。

跟大多数人想的一样，iframe遇到“拒绝访问”首先肯定考虑是document.domain的问题，但是这里父子页面都显式做了降域处理，应该不会。找不出其他原因，只好试了一下，结果屏蔽来屏蔽去调了半天，问题依旧。

后来辗转找到了这篇文章<http://www.cnblogs.com/shouzheng/archive/2008/07/07/1237245.html>，终于解决问题。

问题产生的原因是IE的速度比较慢，在iframe还没有加载的时候是访问不到它的contentWindow,contentDocument之类的对象的，所以报“拒绝访问”。解决的办法很简单，加一个轮询，如果iframe的document.readyState == ‘complete’，再进行操作。

猜测：其他的浏览器一定不出问题么？如果加载一个超级大的文件？还是原理不一样？（如果也有同样的问题的话，其它浏览器可以直接监听iframe的onload事件。）