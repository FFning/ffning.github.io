---
title: '[译]更快构建DOM：预解析，异步，延迟，预加载'
date: 2017-11-20 18:09:21
tags: [preload, 预加载, 翻译, JS]
---
> 原文链接：[Building the DOM faster: speculative parsing, async, defer and preload](https://hacks.mozilla.org/2017/09/building-the-dom-faster-speculative-parsing-async-defer-and-preload/)

在2017年，保证我们的页面可以快速加载的工具包括从压缩，资源优化到缓存，CDN，代码分割和tree shaking(指消除JavaScript上下文中无用代码)的所有内容。然而，即使你对上面提到的概念还不熟悉或者不知道如何开始，你也可以通过一些关键词和精细的代码结构使性能得到巨大的提升。
新的web标准[`<link rel="preload">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content)，允许你更快的加载关键资源，这个月(2017年9月)的晚些时候，Firefox就可以实现。你现在就可以在Firefox的[Nightly](https://www.mozilla.org/en-US/firefox/57.0a1/releasenotes/)或者[Developer](https://www.mozilla.org/en-US/firefox/developer/)版本尝试了，与此同时，这也是一个回顾基本原理，深入了解与DOM解析相关的性能问题的机会。
<!-- more -->对每一个web开发者来说，理解浏览器内部运行机制都是最强大的开发工具。我们将考虑浏览器是如何解析你的代码以及怎样通过预解析更快的加载页面，分解`defer`(延迟)和`async`(异步)是怎样工作的，以及你怎样利用新的关键词`preload`(预加载)。
## **构建模块** ##
HTML描述了一个页面的结构。为了理解HTML的含义，浏览器首先需要把它转换成自己理解的格式——[文档对象模型](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)，或者说DOM。浏览器引擎有一个特殊的代码段叫解析器，用来转化数据格式。一个HTML解析器用来把数据从HTML格式转化为DOM。
HTML中的嵌套，将不同的标签定义为父子关系。在DOM中，对象被连接成树形数据结构用来获取关系。每一个HTML标签都对应树形结构中的一个DOM节点。
浏览器一点一点的构造DOM。一旦有了第一个代码块，它就开始解析HTML，在树形结构中增加节点。
![alt](http://km.midea.com/uploads/imgs/aca970d28260.gif)
DOM有两个作用：它是HTML中的文档的对象表示，也充当连接页面和外部环境的接口，像是通过JavaScript。当你调用document.getElementById()，返回的元素就是一个DOM节点。每一个DOM节点都有许多方法供你获取或改变它，用户也可以看到对应的变化。
![alt](http://km.midea.com/uploads/imgs/46a776a7668a.gif)
基于页面的CSS样式被映射到CSSOM([CSS Object Model,CSS对象模型](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Object_Model))上.这很像DOM，但只是对于CSS来说，而不是HTML。和DOM不同的是，它不能被递增的构建。因为CSS规则可以相互覆盖，所以浏览器引擎必须通过复杂的计算得出CSS代码是如何应用到DOM上的。
![alt](http://km.midea.com/uploads/imgs/f4f5189c2474.png)
## **&lt;script&gt;标签的历史** ##
当浏览器构建DOM的时候，如果发现HTML里有`<script>...</script>`标签，就必须立即执行。如果是引入的外部脚本，就必须先下载脚本。
在过去，为了执行脚本，必须暂停解析工作。只有在JavaScript引擎执行完脚本代码后才能重新开始解析。
![alt](http://km.midea.com/uploads/imgs/0ce5981cce4a.png)
为什么一定要停止解析工作呢？因为脚本是可以改变HTML及其产物DOM的。脚本可以通过`document.createElement()`增加节点以改变DOM结构。为了改变HTML，脚本可以通过臭名昭著的`document.write()`函数来增加内容。说它是臭名昭著的是因为它改变HTML的方式会影响HTML进一步的解析。例如，这个函数可以插入一个打开的注释标签使得余下的HTML无效。
![alt](http://km.midea.com/uploads/imgs/025daeede7c3.gif)
脚本也可以查询一些关于DOM的东西，如果这时DOM还在被构建，就会返回一些意想不到的结果。
![alt](http://km.midea.com/uploads/imgs/123bbf4fbf52.png)
`document.write()`是一个遗留的函数，它会以意想不到的方式破坏你的页面，所以即使浏览器仍然支持这个函数你也不应该使用它了。出于这些原因，浏览器已经有了成熟且复杂的技术来应对我接下来要说的脚本阻塞所带来的性能问题。
## **那么关于CSS呢？** ##
JavaScript会阻塞解析是因为它可以改变文档。CSS是不能改变文档的，所以它看起来没有阻塞解析的理由，是吗？
然而，假设一段脚本查询的样式信息还没有被解析会怎样呢？浏览器不知道脚本怎样执行，它可能想要请求依赖于样式表的东西，像是DOM节点的background-color属性，或者它可能希望直接访问CSSOM。
![alt](http://km.midea.com/uploads/imgs/5c8ca21a52be.png)
正因如此，CSS是否会阻塞解析取决于外部样式表和脚本在文档中的顺序。如果在文档中，外部样式表在脚本的前面，DOM对象和CSSOM对象的构建就会干扰到彼此。当解析过程遇到了script标签，DOM的构建就无法继续进行，直到JavaScript执行完，而JavaScript的执行要等到CSS完成下载，解析并且CSSOM可访问以后。
![alt](http://km.midea.com/uploads/imgs/fd90c393c61f.png)
另一件需要考虑的事就是即使CSS不会阻塞DOM的构建，它也会阻塞渲染。浏览器直到DOM和CSSOM都准备好以前是不会显示任何东西的，因为离开了CSS的页面通常是无法使用的。如果浏览器把一个没有CSS的凌乱的页面展示给你，几分钟后又突然进入了一个有样式的页面，移动的内容和视觉上突然的变化会造成很糟糕的用户体验。
[codepen示例](https://codepen.io/micikato/pen/JroPNm)
这种糟糕的用户体验有一个名字——Flash of Unstyled Content(FOUC)
为了避免这些问题，你应该以尽可能快的实现CSS为目标。还记得流行的最佳实践“styles at the top, scripts at the bottom”吗？现在你知道为什么了！
## **回到未来——预解析** ##
每当遇到脚本解析都会暂停，这意味着每一个你载入的脚本都会延迟链接到HTML中其他资源的发现。
如果你有少数脚本和图片需要载入，例如——
```html
<script src="slider.js"></script>
<script src="animate.js"></script>
<script src="cookie.js"></script>
<img src="slide1.png">
<img src="slide2.png">
```
-过程曾经是这样的：
![alt](http://km.midea.com/uploads/imgs/2182fdd20664.png)
这在大约2008年IE提出了“the lookahead downloader”的概念时发生了改变。这是在同步脚本执行时可以继续下载需要的文件的一种方法。Firefox， Chrome和Safari也紧随其后，现如今，大部分浏览器都以不同的名称在使用这项技术。Chrome和Safari有“预扫描”，Firefox——“预解析”。
总的思想就是：即使执行脚本的时候构建DOM是不安全的，但你仍然可以解析HTML来查看需要检索的其他资源。被发现文件会被加入一个列表并在后台开始并行下载。等到脚本执行完毕，这些文件可能也已经下载完成。
现在上面的例子的瀑布图看起来更像这样：
![alt](http://km.midea.com/uploads/imgs/c135d166d597.png)
这种触发下载请求的方式被称为“预测”，因为脚本还是有可能改变HTML的结构(还记得`document.write`吗？), 造成的结果就是浪费了预测工作。虽然这是可能的，但并不常见，这也是预解析仍可以给性能带来很大提升的原因。
在其他浏览器只是通过这种方法预加载链接的资源时，对FireFox来说，HTML解析器运行构建DOM树的算法也是预测性的。有利的一面是当预测成功时，就不需要重复解析文件中构成了DOM的一部分。不好的一面是一旦预测失败， 就会造成更多工作的浪费。
## **(预)加载的东西** ##
这种资源加载的方式给性能带来了重大的提升，并且你不需要任何特殊的操作就可以利用它的优势。然而，作为web开发人员，知道预解析的工作原理可以帮助你更充分的使用它。
可以被预加载的东西在不同的浏览器之间是不一样的。所有主流浏览器都会预加载：
- 脚本
- 外部CSS
- 以及来自`<img>`标签的图片

Firefox还会预加载video元素的`poster`属性，而Chrome和Safari会预加载内联样式的`@import`规则。
浏览器可以并行下载的文件数量是有限制的。浏览器之间限制的差异取决于很多因素，比如你是从同一个还是从多个服务器下载全部文件，你使用的是HTTP/1.1协议还是HTTP/2协议。为了尽可能快的渲染页面，浏览器通过给每一个文件分配优先级优化下载。计算优先级的复杂方法是以资源类型，标记位置以及页面渲染进度为基础的。
在进行预解析时，浏览器不会执行内部的JavaScript代码块。这意味着它不会发现任何脚本注入资源，这些资源就很可能会排在读取队列的最后。
```javascript
var script = document.createElement('script');
script.src = "//somehost.com/widget.js";
document.getElementsByTagName('head')[0].appendChild(script);
```
你应该让浏览器更轻易的尽快读取重要的资源。你可以将这些资源放在HTML标签里或者将加载的脚本内联到文档的前面。然而，你有时希望一些不重要的资源可以晚些加载，如果这样，你可以通过文档后面的JavaScript加载它们而避免预解析。
你也可以看一下这个针对预解析来优化页面的[MDN指南](https://developer.mozilla.org/en-US/docs/Web/HTML/Optimizing_your_pages_for_speculative_parsing)。
## **延迟和异步** ##
尽管如此，同步的脚本阻塞解析仍然是一个问题。并且不是所有的脚本对用户体验来说都是一样重要的，比如那些为了跟踪和分析的。解决的方案？尽可能异步加载不重要的脚本。
[`defer`和`async`属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#Attributes)的引入就是给开发人员提供一个方法来告诉浏览器哪些脚本需要异步处理的。
这两个属性告诉浏览器在“后台”加载脚本的时候可以继续解析HTML，然后在加载完成后运行脚本。通过这种方式，脚本的下载就不会阻塞DOM构建和页面渲染。结果就是：用户可以在所有脚本加载完成前看到页面。
`defer`和`async`的不同是它们开始运行脚本的时间。
`defer`比`async`先引入。它在解析完全结束时开始运行脚本，但发生在[DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded)事件之前。这可以保证脚本按照它们在HTML中出现的顺序运行且不会阻塞解析。
![alt](http://km.midea.com/uploads/imgs/f2e7544818e8.png)
`async`脚本在它们下载完成之后的第一时间运行，这发生在window的[`load`](https://developer.mozilla.org/en-US/docs/Web/Events/load) 事件之前。这意味着`async`脚本可能(并且很有可能)不按照它们出现在HTML中的顺序运行。这也意味着它们可能中断DOM的构建。
不管他们在哪里被指定，`async`脚本的加载优先级较低。它们通常在所有其他脚本加载后加载，这样就不会阻塞DOM的构建。然而，如果一个`async`脚本很快完成下载，它的运行就可能阻塞DOM的构建和所有之后才完成下载的同步脚本。
![alt](http://km.midea.com/uploads/imgs/8e1446ced0ff.png)
<font size="3" color="#999999">注：`async`和`defer`属性只作用于外部脚本，如果没有`src`，它们会被忽略。</font>
## **预加载(preload)** ##
如果你想要推迟处理一些脚本，`async`和`defer`是很棒的，但是那些页面上对用户体验很重要的东西呢？预解析器很方便，但是它只能遵循它们本身的逻辑预加载一小部分资源类型。通常的目标是首先实现CSS，因为它会阻塞渲染。同步脚本总是比异步的拥有更高的优先级。视口中可见的图片应该比那些下面的图片更早下载完成，也包括字体，视频，SVG等。简而言之，这很复杂。
作为作者，你知道哪些资源对页面的渲染是最重要的。它们中的一些经常深藏在CSS或者脚本中，因此浏览器需要很长一段时间才会发现它们。对于这些重要的资源，你现在可以使用`<link rel="preload">`来告诉浏览器你想要尽快加载它们。
你所要写的就是：
``` html
<link rel="preload" href="http://km.midea.com/?/very_important.js" as="script">
```
你几乎可以链接任何东西，然后`as`属性会告诉浏览器需要下载的是什么。一些可能的值有：
- script
- style
- image
- font
- audio
- video

你可以在[MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content#What_types_of_content_can_be_preloaded)查看其余的内容类型。
字体可能是在CSS中隐藏的最重要的东西。它们对在页面中渲染文本是很关键的，但是直到浏览器确定它们会被应用之前，它们都不会被加载。这个检查的过程只会发生在CSS已经被解析，被应用，并且浏览器已经将CSS规则和DOM节点匹配完成后。这发生在页面加载过程中相当晚的时候而且通常会对文本渲染造成不必要的延迟。当你链接字体的时候，可以通过`preload`属性来避免这一问题。
需要注意的一件事是当预加载字体的时候你也需要设置[crossorigin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)属性，即使字体是同源的：
``` html
<link rel="preload" href="http://km.midea.com/?/font.woff" as="font" crossorigin>
```
目前对`preload`特性的支持还很有限，浏览器仍在不断地推进它，你可以在[这里](https://caniuse.com/#search=preload)查看进展。
## **总结** ##
自从90年代以来，浏览器这些复杂的野兽一直在进化。我们已经讨论了一些历史遗留的怪问题和一些web开发方面的最新的标准。以这些作为参考来写代码将会帮助你选择最好的策略达成流畅的浏览体验。
如果你对学习浏览器工作方式感兴趣的话，可以参考其他文章：
[走近Quantum:什么是浏览器引擎？](https://hacks.mozilla.org/2017/05/quantum-up-close-what-is-a-browser-engine/)
[深入了解一个超快的CSS引擎: Quantum CSS (也称 Stylo)](https://hacks.mozilla.org/2017/08/inside-a-super-fast-css-engine-quantum-css-aka-stylo/)