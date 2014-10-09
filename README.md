# WEBVTT 和 TTML, 让HTML5 video有字幕相伴 #

> 对于视频字幕，大家都不会陌生。那对于HTML5的Video（or audio）字幕，又了解多少呢？本文先通过一个在线视频字幕制作工具来一睹Video Caption的风采，再进一步探究**track元素**以及两种字幕标准：**webVTT**和**TTML**。

### 视频字幕制作工具 ###
先来体验一把微软的这个在线[视频字幕制作工具](http://ie.microsoft.com/testdrive/Graphics/CaptionMaker/ "HTML5 Video Caption Maker")吧，点击下 *Load Sample* 按钮，就会在Video区域看到加载的视频了。播放这个视频再暂停，可以在下面的多行输入框里输入一些文字，保存并继续，该条字幕就出现在了右侧的字幕列表里啦~ 选取所需的时间点，重复以上动作插入多条字幕试试吧！另外，可以根据用户的需要，选择字幕的格式为TTML或WebVTT，将制作好的字幕保存成文件（对应的文件格式分别为.ttml和.vtt），并通过track元素引入该文件，嵌套到video元素内（如下代码所示），就可以在视频播放过程中看到刚制作的字幕啦~！

    <video controls src="demo.mp4">
    	<track kind="captions" src="demo.ttml" srclang="en-us" label="English" />
    </video> 

字幕在视频里的效果如下图：[chrome浏览器]
![视频字幕效果](https://raw.githubusercontent.com/lzf0402/videoCaption/master/demo/image1.png)


以上内容，了解了如何快速的制作字幕文件、字幕与时间的紧密关系以及如何在视频中引入字幕文件，下面就进一步来了解下 **track** 元素、**webVTT**和**TTML**吧

## track元素 ##
**track** 元素（对应的[W3C规范](http://www.w3.org/TR/html51/embedded-content.html#the-track-element)），从上面代码可知，是嵌入在video元素（或audio等媒体元素）内的空内容元素，可以为用户针对视频(或音频)提供多种语言或解说词的计时文本文件。一个video标签，可设置**多个** 计时文本轨道，但需要指定一个default轨道。**track** 元素基本属性如下表：
![track 元素基本属性](https://raw.githubusercontent.com/lzf0402/videoCaption/master/demo/image2.png)

下面，通过一个小demo来介绍下**track** 元素的DOM接口：
> 结构：

    <video id="video" controls src="demo.mp4">
		<track src="ds.vtt" id="track" label="ds vtt" kind="captions" srclang="en-us" default="">
	</video>
	<div id="box"></div>

通过JS实现，将vtt文件里设定的每一条字幕同步显示到#box节点里
> 实现：
 
	//页面初始化 
	function init () {
		var trackNode = document.getElementById("track");
		//绑定track节点对象的load事件
		trackNode.onload = showCue(trackNode);
	}

	function showCue(trackNode){
		//track节点对象有ready状态码 0=NONE 1=LOADING 2=LOADED 3=ERROR
		//chrome下是2,ie下'complete'
		if(trackNode.readyState == 2 || trackNode.readyState == 'complete'){
			//通过track节点对象的cuechange事件，实现同步
			trackNode.addEventListener("cuechange", function (){
				//通过track节点对象的track属性 获得文本轨道对象
	        	var track = this.track;        
				//通过文本轨道对象获得当前激活的字幕对象
	        	var activecue = track.activeCues;
				var box = document.getElementById('box');
		        if(activecue.length > 0){
		        	box.innerHTML = '';
		        	for(var i=0;i<activecue.length;i++){
						//在box节点里显示所有激活的字幕文本
		        		box.innerHTML += activecue[i].text;
		        	}
		       	}
		    }, false); 
		}
	}

	window.onload = init; 
   
以上代码展示了在track节点的cuechange事件里，通过其track文本轨道对象获取当前激活的一条字幕。事实上，track文本轨道对象的cues属性能获取到文本轨道的字幕列表textTrackCueList，该列表的每一项都是一个textTrackCue对象，能通过其text属性获取字幕对象的文本，或者通过getCueAsHTML()方法获取字幕对象的html结构。


## WebVTT ##
**WebVTT** 即The Web Video Text Tracks Format，对应的[W3C文档](http://dev.w3.org/html5/webvtt/ "webvtt规范")，目前还不是W3C的标准。WebVTT文件是用来作为视音频内容的字幕的。 在之前的视频字幕制作工具里，应该已经对WebVTT文件格式有了基本的了解。通过下面这个简单的例子，来了解下WebVTT文件的基本语法。

    WEBVTT demo

	00:00.000 --> 00:06.263 id="t0"
	这是demo.vtt文件,并定义了id
	
	NOTE 我是注释,必须以“NOTE”开头，紧跟空格或换一行，上下要空行
	
	00:06.263 --> 00:12.880 id='t1'
	字幕里可带标签：<p style="color:#f00;">我是红P</p>
	字幕也可以是多行


- 一个WebVTT文件必须要以“WEBVTT”开头，后面可以为空或者是空格加字符串。
- 每一条字幕有一个时间段和一或多条文本组成，且上下都需空一行。
- 时间段由1个开始时间点+“-->”+1个结束时间点组成，时间段后面可以跟该条字幕的id。
- 字幕文本可以换行，也可以带有HTML标签，标签上可以设置样式属性。但上诉代码中的“我是红P”并不会以红色显示。
- 注释可以通过“NOTE”开头紧跟空格或换行再加注释内容的形式来添加，内容可以多行，注释也需要上下空行。

当然，以上的例子能满足基本的字幕表现需求。那如果想要让字幕有更多更丰富的效果，又该如何来设置呢？以chrome37为例

##### 1.对齐 #####
字幕文本默认是居中对齐(middle)，可以通过设置align的值来改变对齐，如下所示：
    
	00:00:00.500 --> 00:00:04.000 align:start
	我是align：start，是文本开头对齐方式
	
	00:00:04.000 --> 00:00:08.000 align:end
	我是align：end，是文本结尾对齐方式
	如果有两行，那两行的结尾对齐

就chrome的实现来看，align设置了非middle值的字幕，只会占据视频宽度的50%，值为start，占据的是右半边，值为end则占据视频的左半边。如果chrome开启了 [shadow Dom](http://www.w3.org/TR/shadow-dom/)的话，可以看到字幕的结构如下图：
![字幕的shadow Dom](https://raw.githubusercontent.com/lzf0402/videoCaption/master/demo/image3.png)

所以，实际的字幕文本是包在一个名为**“cue”的伪元素**中的。而其父节点的style属性里，设置了text-align值为start，且字幕块是绝对定位，width和left都为50%。

##### 2.size #####
可以通过size来设置字幕文本占据视频viewport的宽度比例，如下例子：

    00:00:15.200 --> 00:00:18.700 size:10%
	事实上我被设置了size：10%
	代表占据视频可视区域10%的宽度
在chrome里显示的效果如下：
![字幕的shadow Dom](https://raw.githubusercontent.com/lzf0402/videoCaption/master/demo/image4.png)

##### 3.伪元素 #####
刚刚说到字幕文本实际是包在名为**“cue”的伪元素**中，而这个伪元素，正是我们用来设置字幕文本样式的一个中介。来看下面这个例子：

    00:00:24.000 --> 00:00:28.000 
	样式<v.support>支持</v>么?yes,支持伪元素
	c标签也可用于设置<c.aclass>样式噢
	<v.third.loud>可以在V上设定多个样式
字幕文本可以嵌入V（voice）、C（class）这类标签，同时标签后可以跟相应的样式名称，且可设置多个样式名称。

    ::cue(.aclass){font-size:28px;color:#fe8;background:#ad9;font-weight: bold;font-family:simsun;font-style:italic;}
	::cue(.support){color:yellow;}
	::cue(v[voice="Jack"]){color:blue;border:2px solid #000;padding:0;background:#fff;opacity:0.5;border-radius:5px;}
	::cue(.third){color:red;}
	::cue(.loud){font-size:2em;opacity:0.6;text-shadow:0 0 3px #fc0;border:1px solid #0fc;display: block;padding:10px;}


## TTML ##

