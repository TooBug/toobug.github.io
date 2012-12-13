# JLint——一个sublime的JSLint插件
_2012-12-13 12:50_

先上地址：[https://github.com/TooooBug/JLint](https://github.com/TooooBug/JLint)

### 为什么又造轮子？

Sublime Text 2是一款非常优秀的代码编辑器，前两天出于对它的喜爱，购买了license，从此走上正版的道路。

恰逢最近在项目中看到了一些不是太好的前端代码，就想弄一个JSLint来辅助检查修改一下。在[http://wbond.net/sublime_packages/community#sort-installs](http://wbond.net/sublime_packages/community#sort-installs)上搜索“JSLint”，可以找到三个插件，第一个甚至有2%的用户安装了。但是，仔细看去，这三个插件不是要依赖node就是要依赖java，这让我这种有系统洁癖的人觉得非常不爽。

在找了大半天找不到的情况下，一个想法冒出来了——“逼我么？”

于是有了这个插件。


### 说明

因为是业余作品，花的时间相当少，加之既没有python基础，也不会sublime text插件的开发，全部是现学现卖，因此质量上还有不少问题。

第一个问题是我还不知道如何去打开sublime text 2的控制台，所以在检查完以后需要手工按ctrl+`打开控制台才能看到结果。

第二个问题是不了解其它系统是否有自带的js引擎，所以目前仅支持windows。

第三个问题是windows的脚本宿主（JScript）对连续空行的判断有bug，导致代码中遇到空行后显示的行数不正确。这个坑爹的问题连JSLint作者老道也不准备再搞了，悲剧。

第四个问题是现在代码是同步执行的（还不会搞python子线程的异步执行），文件比较大时会有明显的卡顿现象。

第五个问题是对JSLint的选项还不熟，因此不知道如何配置才最符合工程中的代码要求，这一点我也会自己一边用一边调整。


### 结束

没有了，慢慢改进吧，Node的出现极大地丰富了前端工具，但对不装Node的人来说，还是希望能多一些无依赖的插件可以使用，我也会慢慢去整理一些。
