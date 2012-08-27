# xheditor从剪贴板粘贴图片的实现原理
_2011-11-30 10:08_

[xheditor](http://xheditor.com/)是一个优秀的国产富文本编辑器，最近的（其实我也不知道有多久了）版本推出了一个粘贴剪贴板图片的功能，很是喜爱，好奇了好久，最终不住去看了下源码，基本明白了。

首先，这个功能很新，不支持IE系列浏览器……所以用的是一些新的API。

我手上的代码是1.1.10，所以代码行数以这个版本为准。

一点前置小知识是必备的，那就是对大部分截图软件来说，截图操作是一个把截取的图片转换成指定格式（貌似jpg居多， windows自带的是bmp），然后放入剪贴板的过程。

所以，要粘贴图片，核心就是读取剪贴板。

$$solo_more$$

### 一、读取剪贴板的时机

近几年浏览器在拼标准、拼性能的时候，其实也在暗里拼安全，为了防止某个网页一打开就自动把你剪贴板的内容传网上去（IE6可以做到），现代浏览器不允许随时读写剪贴板。

浏览器为我们提供了onpaste事件（粘贴），读取剪贴板数据仅能在该事件发生时在事件处理程序中进行。

因这里理解就好，所以就不贴代码。

### 二、图片数据

之所以IE不能粘贴图片，就在于IE没有处理二进制文件的机制，它只能处理文本。

在现代浏览器中，则有关于文件的API，这里用到的就是File Reader，它主要用来处理二进制文件数据。

首先，在xheditor的第1922行，有一个cleanPaste函数，它的作用就是读取剪贴板的数据，然后放入编辑器中，具体见代码注释。

	var clipboardData,items,item;//for chrome

	//ev是事件，其实大家都喜欢用e或者evt或者event
	//这句主要是确定剪贴板中是否有图片
	if(ev&&(clipboardData=ev.originalEvent.clipboardData)&&(items=clipboardData.items)&&(item=items[0])&&item.kind==’file’&&item.type.match(/^image\//i)){
		var blob = item.getAsFile(),reader = new FileReader();//blob就是剪贴板中的二进制图片数据

		//定义fileReader读取完数据后的回调
		reader.onload=function(){
			var sHtml=’<img src=”‘+event.target.result+’”>’;//result应该是base64编码后的图片
			sHtml=replaceRemoteImg(sHtml);//这里执行了一个将base64上报到服务器，然后将图片url从base64编码的图片数据换成上传后的图片地址
			_this.pasteHTML(sHtml);//这里应该是关于光标和插入代码的具体操作
		}

		reader.readAsDataURL(blob);//用fileReader读取二进制图片，完成后会调用上面定义的回调函数
		return false;
	}

好吧，本来还想写下中间上报到服务器的代码的，想了想觉得也没什么很特别的地方。最多是把base64数据当成普通URL上报，然后服务端再解码成图片。

嗯。就这样，当笔记了。