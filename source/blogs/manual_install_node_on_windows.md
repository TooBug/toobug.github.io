Title: 在Windows上手工安装Node.js
Date: 2013-12-19 20:55:00
Tags: 工具 Node Node.js windows

时间已经到了2013年末，如果在这个时间点，你还没有接触过Node.js，那可能真的是有点跟不上时代步伐了。技术发展总是很快的，Node项目诞生不过才4年，在windows平台出现也才2年多的时间，却已如星星之火点起了燎原之势……

没错，Node出现超过4年了，但在windows平台才2年多的历史，可见支持windows多少是个有点艰难的决定，好在背后有微软撑腰，一切顺利。至于说到这个标题，手工安装嘛，如果去搜索一下，也能找到一些文章，不过大部分已经是0.6之前版本的故事了，那时候Node还没有给windows的安装包，也没有集成包管理工具npm，用起来还是挺不方便的，而如今，只要去<http://nodejs.org>上下载对应的.msi文件再安装就万事大吉了。

不过，微软家的东西总是会有些意料之外的状况，.msi也不例外……

<!-- $$solo_more$$ -->

这周给咱组里的[skpping](http://skpping.cdc.im)妹子安装Node的时候，就出现了这么一幕：

![Node .msi安装出错](/images/manual_install_node_on_windows_1.jpg)

首先我想到的方案自然是修改.msi的安装，但经过近一个小时的努力，最终宣告失败，无奈之下，只能选择手工安装。

## 安装Node

其实手工安装Node是一件很简单的事情，我们的目标是在命令行中可以运行`node`命令即可。首先，我们需要建立一个用来存放node（以及接下来会讲到的npm）的目录，比如我把它建在了`C:\node`。然后，为了能在命令行中直接使用`node`，需要将这个目录添加到`path`环境变量中。具体的方案就不讲了，搜索一下可以找到一堆教程。

再接下来，从Node.js官网下载二进制文件，即`node.exe`，然后将它放到`C:\node`，重新启动命令行，即可使用`node`命令了。

![Node安装成功](/images/manual_install_node_on_windows_2.png)

## 安装npm

其实Node官方推出windows版的.msi安装文件，除了会处理好环境变量之外，另外一个最重要的意义就是npm了。什么？你不知道什么是npm？好吧，比较官方的说法是，它是Node的模块管理软件，民间一点的说法就是，玩Node必备的东西，没有的话会让你玩得生不如死。

目前Google搜索windows手工安装npm，排名第一的是[这篇](http://www.cnblogs.com/seanlv/archive/2011/11/22/2258716.html)，但在这个时间点，其实有点误导人了。这篇文章中说的方法是需要下载npm的源码，然后执行手工安装。但按照npm的[官方文档](https://npmjs.org/doc/README.html)，只需要下载二进制包，解压即可使用。

具体的方法，先到<https://npmjs.org/doc/README.html>下载二进制包，然后解压到我们刚刚建立的`C:\node`目录，注意解压后的`npm.cmd`和`node_modules`都要在`C:\node`目录下，而不是更深的子目录。再重启一次命令行，就可以使用npm啦！

![npm安装成功](/images/manual_install_node_on_windows_3.png)

## 小结

嗯，至此手工安装就完成啦。很水是不是？很水是不是？好吧，我也觉得很水，为了让它水得不那么彻底，我写了一个半自动化的脚本来完成所有的事情。直接上代码好了。

	mkdir download
	wget http://nodejs.org/dist/v0.10.23/node.exe -O download\node.exe
	wget http://nodejs.org/dist/npm/npm-1.3.21.zip -O download\npm.zip
	unzip -o download\npm.zip -d download
	echo Y | del download\npm.zip
	mkdir c:\node
	move download\* c:\node
	move download\node_modules c:\node
	echo Y | del download
	wmic ENVIRONMENT where "name='path' and username='<system>'" set VariableValue="%path%;c:\node"

把以上代码存为`setup.cmd`文件，然后双击，就可以啦。咦，没反应是不是？哦，对，忘了，还有两个依赖：wget和unzip，一个用于下载文件，一个用于解压。完整的文件可以在[这里](http://url.cn/PU9VLO)下载。

最后的最后，这个代码会依赖wmic，在XP下第一次使用会自动安装。Win7就没测了，印象中是默认支持的。

> 补充说明一下，文章中的图是盗来的，所以有神马乱入的名字，不一致的版本号之类的问题……

