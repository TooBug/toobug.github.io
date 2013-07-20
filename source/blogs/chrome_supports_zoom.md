# Chrome也支持zoom了
_2011-12-21 19:59_

查阅网上的中文资料，几乎每篇文章都在说，zoom是IE专有的属性，但事实是，chrome也支持。

Demo：<http://jsfiddle.net/toobug/9drpy/1/embedded/result/>

所以在chrome下放大元素除了用css3之外，也可以用zoom这样简单的方法了。

> 2013-07-20更新1：现在网上再查已经可以查到有很多资料在讲chrome支持zoom属性了。顺带提一下，CSS3中用transform的写法：`transform:scale(0.5);`，但这个缩放是以中心点为原点来进行的，可以通过`transform-origin:top left`来修正。

> 2013-07-20更新2：在IE中zoom被用于激活hasLayout属性的作用远大于它本身的缩放作用。但事实上，zoom在进行缩放时，部分低版本IE下（比如IE7）在遇到`position:relative`的元素时会有各种定位漂移的bug，因此，慎用为好。