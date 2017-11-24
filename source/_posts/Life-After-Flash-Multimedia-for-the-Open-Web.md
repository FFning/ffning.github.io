---
title: '[译] 后 Flash 时代：开放网络下的多媒体技术'
date: 2017-10-11 18:32:33
tags: [Flash, 动画, 多媒体, 翻译]
---
> 原文地址：[Life After Flash: Multimedia for the Open Web][2]

Flash在十多年来提供了视频、动画、互动网站以及面向对数十亿用户的广告，但现在它已经逐渐消失了。Adobe将在2020年停止对Flash的支持。Firefox和Chrome也不再支持Flash。接下来将会有大量的开放标准可以做到Flash能做的，甚至更好。
### **真正开放的多媒体**
Flash承诺提供一个统一的平台，用于构建和实现交互式的多媒体网站，并且在极大程度上也实现了这一设想。但技术从来都不是真正意义上的开放和易用的，并且Flash Player在移动设备上运行时会造成了大量的资源消耗。现有的开源技术完全可以替代Flash并且做的更好。如果你真的想要构建前沿的交互式网站，无论是网站[动画][3]，[游戏][4]还是[视频][5]，这些都将是你需要掌握的技术。<!--more-->
### **web动画**
#### CSS
[CSS动画][6]是相对比较新的技术，但它也是最简单的开始构建网站动画的方法。CSS是用来控制网站样式的规则，这些规则规定了布局，字体，颜色等。随着CSS3的发布，动画已经融入了新的标准，作为一个开发者，你需要告诉浏览器如何显示动画。CSS的可读性是很高的，也就是说它是基本上是按照字面意思执行的，比如说“animation-direction”属性，就是指动画播放的方向(正向/反向)。
现在你可以使用CSS创建平滑流畅的动画，创建[关键帧][7],调整时间和动画的不透明度等都是很简单的，而且所有的动画都与你平时用CSS处理的文本，图片，容器等一起运行。
即使你对编程语言不熟悉，你也可以只通过CSS实现动画。网上也有许多[开源项目][8]供你把玩。Mozilla也创建并维护了详尽的[CSS动画文档][9]。大多数开发人员都建议使用CSS来构建简单项目的动画，而更复杂的项目则使用JavaScript。
#### JavaScript
开发人员从早期就在使用JavaScript构建动画了，基本的mouseover的脚本就已经存在20多年了，现在利用JavaScript并结合HTML5的[`<canvas>`][10]元素，我们可以实现更多神奇的效果。就算只是很简单的脚本，也可以实现[很好的效果][11]。使用JavaScript，你可以绘制图形，改变颜色，移动和更改图片，改变动画透明度等。JavaScript可以使用[SVG][12] (scalable vector graphics)格式生成动画，这些动画实际上是基于数学的而不是加载和渲染出来的。所以它们在任意比例下都可以保证清晰(人如其名啦)并且是可控的。SVG提供了抗锯齿渲染，图案填充，渐变填充，复杂的滤镜效果，对任意路径的剪切，文本功能以及动画。当然了，它是W3C的开放标准，而不是一个封闭的二进制文件。使用SVG、JavaScript和CSS3，开发人员就可以创建令人印象深刻的[交互式动画][13]了，不需要任何专门的格式或播放器。
JavaScript动画可以非常精简，包括跳过，停止，暂停，后退和减速。它也可以是交互式的，通过编程实现对鼠标的点击操作和滚动操作的响应。使用JavaScript构建web动画的新的[Web Animations API][14]，可以让你更大程度的控制关键帧和元素从而调整动画效果，但是它还处于早期开发试验阶段，所以某些特性没有得到全部浏览器的支持。
此外，JavaScript动画可以用来响应字段的输入，提交表单和键盘事件，这使他成为了构建web游戏的完美选择。
### **web游戏**
曾几何时，Flash在web游戏中是出于统治地位的，它的学习，使用和传播都很简单，同时也很健壮，可以实现数百万人的大型多人在线游戏。但是今天我们使用JavaScript, HTML5, WebGL 和 WebAssembly可以达到不比Flash差的体验。结合现代浏览器和开源框架，我们就可以[构建3D动作射击游戏，RPGs，冒险游戏等等][15]。事实上，你现在甚至可以通过[WebVR][16]和[A-frame][17]等技术，在网页上创建完全沉浸式的虚拟现实体验。
web游戏依赖于开源框架和平台的生态系统去运行。从视觉到控制，从音频到网络，每一部分都扮演着重要的角色。MDN有一套详尽的正在应用中的[技术列表][18]，下面是其中的一些和它们的用途：
**[WebGL][19]**
让你能够在页面中创建高性能的，硬件加速的3D/2D图形，它是在支持Web端的基础上基于[OpenGL ES][20] 2.0实现的。WebGL 2则更进一步，实现了OpenGL ES 3.0对浏览器的支持。
**[JavaScript][21]**
JavaScript是在Web端运行的编程语言，它在浏览器中运行良好并且速度越来越快。它已经被用来构建了成千上万的游戏，各种新的游戏框架也层出不穷。
**[HTML audio][22]**
[`<audio>`][23]标签可以很方便的实现简单的音效和音乐的播放。如果需要更多的相关知识，可以参照[Web Audio API][24]获得真正的处理音频的能力。
**[Web Audio API][25]**
通过JavaScript代码实现对音频的回放、合成等处理的快速的API，让你在创建非常棒的音效的同时还可以实时播放和操控音乐。
**[WebSockets][26]**
WebSocket API允许你将你的应用或站点连接到一个服务器，并实时地传输数据。适用于多人回合制的游戏或聊天服务等。
**[WebRTC][27]**
WebRTC是一个可以用在视频聊天，音频聊天或P2P文件分享等Web App中的API。它适用于需要更低延迟的实时的多玩家游戏。
**[WebAssembly][28]**
HTML5/JavaScript的游戏引擎已经比以往好很多了，但是它们依然不如原生应用的性能。[WebAssembly][29]有希望给Web应用带来近乎原生的游戏引擎的性能。该技术允许浏览器[运行编译后的c/c++代码][30]，包括使用像[Unity][31]和[Unreal][32]这样的引擎所制作的游戏。
WebAssembly使web游戏具有[多线程的优势][33]，开发者在不损害安全的前提下可以开发出在web端的运行速度接近于原生代码的惊人的3D游戏。这对于游戏和开放网络来讲都是巨大的突破。这意味着开发人员可以为任何能够访问web的计算机或系统构建游戏。因为游戏运行在浏览器中，所以更容易集成为多人在线的模式。
除此之外，web游戏也有许多其他的[HTML5/JavaScript游戏引擎][34]。这些引擎负责基础，比如物理学和控制装置，提供一个框架/世界给开发人员来构建游戏。包括像是[atom][35]和[Quick][36]这样的轻量级的快速的2D引擎，也包括像是[WhitestormJS][37]和[Gladius][38]这样的功能全面的3D引擎，这样各有利弊的游戏引擎有很多可供开发人员选择。同样的是，他们都可以在不安装插件的情况下在现代浏览器上运行游戏，而且大多数游戏都可以在不太强大的硬件上运行，这意味着你可以传播给更多的用户。事实上，为web编写的游戏可以在平板电脑、智能手机甚至智能电视上运行。
MDN有大量关于[构建web游戏的文档][39]，以及一些关于使用[纯JavaScript][40]和[Phaser游戏框架][41]构建游戏的教程。是一个开始学习web游戏开发的好地方。
### **Video**
大多数视频服务已经利用web技术和开放的解码技术切换到基于HTML5的服务流，其他的则坚持使用[基于Flash的FLV或FV4解码技术][42]。综上，Flash视频格式依赖于软件渲染，这对web浏览器和移动平台压力较大。现代视频解码器可以通过硬件渲染实现视频播放，很大程度地提高了响应速度和效率。不太好的消息是，从Flash切换到HTML5只有一种方法：重新编码你的视频。可以通过像是[FFmpeg][43]和[Handbrake][44]这样的免费转换器将您的源视频转换为支持HTML5的格式。
Mozilla正在积极帮助构建和提升HTML5的友好程度以及[开源的视频格式WebM][45]。WebM基于[Matroska][46]容器，使用[VP8][47]和[VP9][48]视频解码器以及[Vorbis][49]或[Opus][50]解码器。
当你的媒体文件已经被转换成一种HTML5支持的格式，你就可以在你的网站上重新发布你的视频。HTML5有内置的媒体控制，所以不需要安装任何播放器。使用起来非常简单，只需使用一行HTML代码:
```html
<video src="videofile.webm" controls></video>
```

请记住，原生的媒体控件在不同的浏览器之间是有差异的。因为它们是基于HTML5的，所以你可以用[CSS控制样式][51]，然后用JavaScript与你的视频关联起来。这意味着您可以在不同的浏览器上构建看起来一样的可访问的并且有个人品牌的播放控件。
HTML5还可以处理[Media Source Extensions(MSEs)][52]标准下的自适应流。虽然他们很难自行设置，但是你可以使用[像Shaka Player][53]和[JW Player][54]这样预先封装好的播放器来处理细节。
MDN的开发人员已经创建了一份详尽深入[将Flash视频转换为HTML5视频的方法][55]，并提供了更多关于该过程的细节，幸运的是，这也没有看起来这么困难。
### **Flash的未来**
web的未来是开放的([希望如此][56])，Flash尽管是一个优秀的充满创意的工具，但还不够开放。值得庆幸的是，我们有许多开源工具可以比Flash做得更好。但这些技术仍处于早期阶段，创建动画、交互性的网站和web游戏需要一些[编码知识][57]，需要你了解的东西就在那里等着你去发掘。
开放的网络技术有希望比Flash更好，并且我们每个人都可以拥抱它。

  [2]: https://hacks.mozilla.org/2017/08/life-after-flash-multimedia-for-the-open-web/
  [3]: https://developer.mozilla.org/en-US/docs/Web/API/Animation
  [4]: https://developer.mozilla.org/en-US/docs/Games
  [5]: https://developer.mozilla.org/en-US/docs/Plugins/Flash_to_HTML5/Video
  [6]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations
  [7]: https://en.wikipedia.org/wiki/Key_frame
  [8]: https://daneden.github.io/animate.css/
  [9]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations
  [10]: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Basic_usage
  [11]: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Basic_animations
  [12]: https://developer.mozilla.org/en-US/docs/Mozilla/Mozilla_SVG_Project
  [13]: http://slides.com/sdrasner/svg-can-do-that#/
  [14]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API
  [15]: https://developer.mozilla.org/en-US/docs/Games/Introduction
  [16]: https://mozvr.com/
  [17]: https://aframe.io/
  [18]: https://developer.mozilla.org/en-US/docs/Games/Introduction
  [19]: https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API
  [20]: http://www.khronos.org/opengles/
  [21]: https://developer.mozilla.org/en-US/docs/Web/JavaScript
  [22]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio
  [23]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio
  [24]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
  [25]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
  [26]: https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
  [27]: https://developer.mozilla.org/en-US/docs/Glossary/WebRTC
  [28]: https://research.mozilla.org/webassembly/
  [29]: https://research.mozilla.org/webassembly/
  [30]: https://hacks.mozilla.org/2017/07/webassembly-for-native-games-on-the-web/
  [31]: https://unity3d.com/
  [32]: https://www.unrealengine.com/en-US/what-is-unreal-engine-4
  [33]: https://hacks.mozilla.org/2017/07/webassembly-for-native-games-on-the-web/
  [34]: https://github.com/bebraw/jswiki/wiki/Game-Engines
  [35]: https://github.com/nornagon/atom
  [36]: https://github.com/diogoschneider/quick
  [37]: https://github.com/WhitestormJS/whs.js
  [38]: https://github.com/gladiusjs/gladius-core
  [39]: https://developer.mozilla.org/en-US/docs/Games/Introduction
  [40]: https://developer.mozilla.org/en-US/docs/Games/Tutorials/2D_Breakout_game_pure_JavaScript
  [41]: https://developer.mozilla.org/en-US/docs/Games/Tutorials/2D_breakout_game_Phaser
  [42]: https://en.wikipedia.org/wiki/Flash_Video
  [43]: http://ffmpeg.org/
  [44]: https://handbrake.fr/
  [45]: https://www.webmproject.org/
  [46]: https://www.matroska.org/technical/whatis/index.html
  [47]: https://en.wikipedia.org/wiki/VP8
  [48]: https://www.webmproject.org/vp9/
  [49]: http://www.vorbis.com/
  [50]: http://www.opus-codec.org
  [51]: https://developer.mozilla.org/en-US/Apps/Fundamentals/Audio_and_video_delivery
  [52]: https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API
  [53]: https://github.com/google/shaka-player
  [54]: https://www.jwplayer.com/
  [55]: https://developer.mozilla.org/en-US/docs/Plugins/Flash_to_HTML5/Video
  [56]: https://advocacy.mozilla.org/en-US/net-neutrality/la.org/en-US/docs/Plugins/Flash_to_HTML5/Video
  [57]: https://developer.mozilla.org/en-US/