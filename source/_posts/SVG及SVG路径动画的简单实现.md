---
title: SVG及SVG路径动画的简单实现
date: 2017-11-24 14:41:33
tags: [SVG, 动画]
---
## **SVG**
[可缩放矢量图形（Scalable Vector Graphics，SVG)](https://developer.mozilla.org/en-US/docs/Web/SVG)，是一种用来描述二维矢量图形的XML 标记语言，是一个W3C标准，所以大多数情况它符合HTML的特性，且能够与CSS等其他W3C标准协同工作。通过它的名字也不难看出，在需要保证不同尺寸、不同分辨率的屏幕上的一致体验时，SVG是个很好的选择。
SVG作为一种图像格式，与JPEG、PNG等其他图像格式相比较的优势在于其尺寸更小且可压缩性更强，支持多种工具打开和修改，最重要的是它的可缩放性，保证了SVG图像可在各种尺寸的屏幕上保证图像质量，也可以在任何分辨率下被高质量的打印。与同属二维矢量图形的FLASH相比，SVG是一个W3C标准，基于XML，是开放的，而FLASH是封闭的，基于二进制格式的。
SVG图像是使用大量的包括动画元素、形状元素、容器元素等在内的[SVG元素](https://developer.mozilla.org/en-US/docs/Web/SVG/Element)创建的，这些元素用于构建、绘制矢量图形。[SVG属性](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute)包括动画属性，图形事件属性，文档事件属性等，SVG属性可以用来修改SVG元素，这些属性决定了元素的呈现方式及细节。同时对应不同的SVG元素，也有丰富的[SVG DOM API](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model#SVG_interfaces)，提供了对元素属性的访问及操作元素的方法。
<!--more-->下面通过一个简单实例及代码具体了解一下SVG：
![圆角矩形](https://pic.mdcdn.cn/h5/pic/201711/5a069f4d07302.svg)
对应的SVG文件代码如下
```html
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" width="100%" height="100%" version="1.1">

<rect x="20" y="20" rx="20" ry="20" width="250" height="100" style="fill:red;stroke:black;stroke-width:5;opacity:0.5"/>

</svg>
```
代码的第一行引用了一个外部的 SVG DTD文件。该 DTD 位于 W3C，含有所有允许的 SVG 元素。
SVG 代码以 `<svg>` 元素开始，包括开启标签 `<svg>` 和关闭标签 `</svg>` ,这是根元素。
`xmlns` 属性定义了该 SVG的 命名空间。version 属性可定义所使用的 SVG 版本，width 和 height 属性可设置此 SVG 文档的宽度和高度。
SVG 的 `<rect>`元素 用来创建一个矩形。x 和 y 属性定义矩形左上角的 x 和 y 坐标。rx 和 ry 属性定义圆角在水平及垂直方向上的半径。width 和 height 属性控制矩形的宽高。style属性中的fill值设置形状内的颜色，stroke 和 stroke-width 控制如何显示形状的轮廓。
## **SVG路径**
如果说SVG中最强大的元素，那么就非[`<path>`(路径元素)](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/path)莫属了。path元素是用来定义形状的通用元素。所有的基本形状都可以用path元素来创建。
```html
<!-- A right-angled triangle -->
<path d="M10 10 L75 10 L75 75 Z" />
```
你可以想象一支在屏幕上绘画的虚拟的画笔，而path元素中的属性就用来控制这支笔。上面的示例代码中，首先画笔画了一条以为`(10,10)`(M10 10)为起始位置，以`(75，10)`(L75 10)为终止位置的直线，接着又画一天直线至`(75，75)`(L75 75)，最后的z表示画笔回到起始坐标点形成闭合路径。
其中的 M 和 L 属于绘图指令，M规定了起始点的位置，L 则表示画一条直线至指定位置，还有许多其他的绘图指令如A(圆弧)、Q(二次贝塞尔曲线)、C(三次贝塞尔曲线)等等，这些丰富的指令可以帮助你在SVG中创建复杂精致的矢量图形。
### **SVG路径与CSS**
让静态的SVG路径元素动起来，我们首先要利用SVG的两个属性：[`stroke-dasharray`](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/stroke-dasharray) 和 [`stroke-dashoffset`](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/stroke-dashoffset)。

![alt](http://km.midea.com/uploads/imgs/f84f9c4554fd.jpg)

属性`stroke-dasharray`可以控制图案描边路径的样式，在你不想要连续的直线作为路径时，你可以使用这个属性得到不同样式的虚线。
由于SVG图像是浏览器DOM的一部分，而`stroke-dasharray`作为控制外观的元素，也可以用在CSS中。类似地，`stroke-dashoffset`属性(指定了dash模式到路径开始的距离)也可以使用CSS来控制。这两个SVG属性组合在一起，可以生成SVG动画，使路径看起来正在逐渐被绘制。
以这个二次bezier曲线为例:

![](https://pic.mdcdn.cn/h5/pic/201711/5a0939fd2b0fa.svg)
```html
<path fill="transparent" stroke="#000000" stroke-width="5" d="M10 80 Q 77.5 10, 145 80 T 280 80" class="path"></path>
```

要使这条路径动画化，就像它在屏幕上逐渐和平滑地绘制一样，我们必须使`stroke-dasharray`属性的值等于路径长度，也就是说虚线曲线上每一个实线段和缝隙的长度都等于整个路径的长度。
接下来，我们将使`stroke-dashoffset`属性设置为0。这将使路径在屏幕上显示为一个实曲线(我们本质上是在看虚线的第一个实线段)。通过设置与曲线长度相等的`stroke-dashoffset`，我们最终会得到一条看不见的曲线。
现在，结合CSS3的animation改变`stroke-dashoffset`属性，可以使曲线逐渐出现在屏幕上，如下：

![](https://pic.mdcdn.cn/h5/pic/201711/5a0940b5e94eb.svg)

```css
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" 
"http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<svg dth="300px" height="175px" version="1.1" xmlns="http://www.w3.org/2000/svg">
  <style type="text/css" >
    .path {
      stroke-dasharray: 320;
      stroke-dashoffset: 0;
      animation: dash 1s linear infinite;
    }
    @keyframes dash {
      from {
        stroke-dashoffset: 320;
      }
      to {
        stroke-dashoffset: 0;
      }
    }
  </style>
  <path fill="transparent" stroke="#000000" stroke-width="5" d="M10 80 Q 77.5 10, 145 80 T 280 80" class="path"></path>
</svg>
```

如你所见，我们只是在改变dash偏移量以使虚线部分动态的出现，便实现了动画的效果，使用同样的原理结合多条路径可以达到更复杂的动画效果，如下：

![](https://pic.mdcdn.cn/h5/pic/201711/5a09421a24699.svg)

```html
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" 
"http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<svg dth="300px" height="175px" version="1.1" xmlns="http://www.w3.org/2000/svg">
  <style type="text/css" >
    .line1 {
      stroke-dasharray: 340;
      stroke-dashoffset: 40;
      animation: dash 1.5s linear alternate infinite;
    }
    .line2 {
      stroke-dasharray: 320;
      stroke-dashoffset: 320;
      animation: dash2 1.5s linear alternate infinite;
    }
    @keyframes dash {
      from {
        stroke-dashoffset: 360;
      }
      to {
        stroke-dashoffset: 40;
      }
    }
    @keyframes dash2 {
      from {
        stroke-dashoffset: 280;
      }
      to {
        stroke-dashoffset: -40;
      }
    }
  </style>
    
  <path fill="transparent" stroke="#000000" stroke-width="4" d="M10 80 Q 77.5 10, 145 80 T 280 80" class="path"></path>
  <path fill="transparent" stroke="#FF2851" stroke-width="4" d="M10 80 Q 77.5 10, 145 80 T 280 80" class="line2"></path>
  <path fill="transparent" stroke="#000000" stroke-width="4" d="M10 80 Q 77.5 10, 145 80 T 280 80" class="line1"></path>
</svg>
```

`stroke-dasharray`和`stroke-dashoffset`是两个非常强大的属性，可以使SVG路径产生大量的动画效果。
### **沿SVG路径的动画对象**
同样的只是利用SVG和CSS，您可以做的一件更酷的事，使对象或元素沿SVG路径运动。这里又需要了解两个新的可以作用于SVG的CSS属性：
`offset-path`：指定元素的运动路径。
`offset-distance`：定义了元素在路径上运动的距离，单位是数值或百分比。
结合这两个属性，您可以得到这样的动画：
![](https://pic.mdcdn.cn/h5/pic/201711/5a09485974484.gif)

```html
<style type="text/css" >
.ball {
  width: 10px;
  height: 10px;
  background-color: red;
  border-radius: 50%;
  
  offset-path: path('M10 80 Q 77.5 10, 145 80 T 280 80');
  offset-distance: 0%;
  
  animation: red-ball 2s linear alternate infinite;
}
@keyframes red-ball {
  from {
    offset-distance: 0%;
  }
  to {
    offset-distance: 100%;
  }
}
</style>
<svg dth="300px" height="175px" version="1.1" xmlns="http://www.w3.org/2000/svg">
  <path fill="transparent" stroke="#888888" stroke-width="1" d="M10 80 Q 77.5 10, 145 80 T 280 80" class="path"></path>
</svg>

<div class="ball"></div>
```

gif动画效果不流畅，[点此查看codepen示例](https://codepen.io/toptalblog/pen/qXRJeY)
如你所见，我们有一个新的元素`div.ball`，并让该元素沿路径运动。
这个元素可以是任何类型的，一个div，一个图像，文本，等等。使用了`offset-path`和`offset-distance`，您可以为元素提供一个运动路径并控制运动的距离，达到动画的效果。
需要注意的是，`offset-path`和`offset-distance`目前仍属于实验中的功能，[浏览器的兼容性](https://caniuse.com/#search=offset-path)很不乐观。
### **使用JavaScript做SVG动画**
在使用JavaScript对SVG元素做动画时，完全可以把SVG元素当做普通的DOM元素处理，而不再需要将路径长度硬编码在CSS中。运用丰富的第三方SVG动画库，我们也可以更容易地实现上面提到的动画效果。
例如[Snap.svg](http://snapsvg.io/)，不仅让使用JavaScript绘制SVG图形变得更容易，同时只需要调用.animate({})这个API，非常简单。另外一个库[anime.js](http://animejs.com/)，让你只通过[几行代码](https://codepen.io/collection/XLebem/)就可以实现让一个元素沿着SVG路径运动。[Vivus](https://maxwellito.github.io/vivus/)已经为你做了大部分操作来生成复杂的动画，它采取了一种不同的调用方式，仅需要通过配置项的方式去生成SVG路径动画。你只需要用SVG元素的id来定义一个Vivus对象，然后就交给Vivus来处理剩下的事情。
### **扩展阅读**
打造SVG动画的三种途径：[three ways to animate SVG](https://css-tricks.com/video-screencasts/135-three-ways-animate-svg/)
SMIL（Synchronized Multimedia Integration Language）相关知识：[SMIL](https://www.w3.org/TR/REC-smil/)，[A Guide to SVG Animations (SMIL)](https://css-tricks.com/guide-svg-animations-smil/)

 参考文献：
    [A How-to Guide to SVG Animation](https://www.toptal.com/front-end/svg-animation-guide)
    [MDN web docs-SVG](https://developer.mozilla.org/en-US/docs/Web/SVG)
