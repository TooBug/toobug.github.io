# jSound Alpha
_2012-04-06 12:27_

某天突发奇想，咋没有一个关于声音的js库？本来HTML5的声音就很弱，这方面应该有更多的封装才适合做游戏之类的东东。

在网上找了一下，有个叫[soundmanager](https://github.com/nicklockwood/SoundManager)的库，不过它是调用flash的声音能力，当然功能也比较强大。

于是就自己写了一个，目前还只是最最简单的版本，全部代码如下：

$$solo_more$$

	~function(w){
		if(w.jSound)return;
		~function(){
			if (!document.body){
				setTimeout(arguments.callee,50);
				return;
			}
			var elem = document.createElement('audio'),src;
			if (!elem.canPlayType){
				elem = document.createElement('bgsound');
			}
			document.body.appendChild(elem);
			w.jSound={
				play:function(soundSrc){
					src=soundSrc;
					elem.src='';
					elem.src=src;
					if (elem.canPlayType){
						elem.play();
					}
				}
			}
		}();
	}(window);

目前只是封装了IE与非IE的差别，IE下用bgsound，非IE用audio标签，还有比较多的事情要解决：

- 声音重叠（IE下必须播放完后才能重新放同一个声音，即使将src赋空后重新赋值也不行）
- URL请求，目前每点击一次在高级浏览器下会发出两个HTTP请求，原因还没仔细去看，后续看看能否优化成下载后不再发出请求
- 多声音管理，本来刚开始是将jSound写成构造函数的，然后每个对象管理自己的src和页面元素elem，但是试用了一下觉得似乎不太必要又改为了现在全局对象的样子，是否需要单独的对象要再衡量一下
- 播放状态管理，这个估计有困难

> 8月17日更新：已经开源到[GitHub](https://github.com/TooooBug/jSound)，另外许久未打理，估计得过段时间再继续完善。