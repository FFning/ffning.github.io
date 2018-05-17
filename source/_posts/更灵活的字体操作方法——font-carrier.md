---
title: 更灵活的字体操作方法——font-carrier
date: 2017-12-25 18:34:46
tags: [SVG, 字体]
---

事情是这样的，有位同学看了前段时间我发在知乎上的一篇关于svg译文之后发了私信给我问了我一个svg方面的问题，原问题是这样的：

>  想要知道 font怎么样通过js或者php 在线转换成svg的path

没见过世面的我看到这句话时整个人是懵的，font在线转svg？？

![](http://km.midea.com/uploads/imgs/0f6f4e280229.png)

仔细问了一下，大概的需求是需要把输入的字在线转成svg图像，可以方便的调整位置颜色并提供下载，了解需求之后，没见过世面的我表情还是…

![](http://km.midea.com/uploads/imgs/0f6f4e280229.png)

因为是第一次被人私信问问题所以还是认真的找了一下资料的，竟让给我找到了…

蹡蹡~就是这个—— [font-carrier](https://github.com/purplebamboo/font-carrier)，官方介绍是'一个功能强大的字体操作库，使用它你可以随心所欲的操作字体'。大概的功能包括解析已有字体文件，导出主流格式的字体文件，生成某个字的svgXML代码(bingo!)。
<!--more-->
![alt](http://km.midea.com/uploads/imgs/49dc5234e602.gif)

进一步沟通之后发现这位同学竟然也是用这个库的时候遇到了问题(他遇到的问题很基础并且和主题无关，这里就不详谈了)，解决了以后大概看了一下文档，发现这个库(虽然快两年没有更新了但)还是挺有意思的，所以在这里也就部分功能总结一下，大概归纳了一下可能会应用到的场景，仅供参考~~（[<font color="#999">戳这里查看详细API</font>](https://github.com/purplebamboo/font-carrier/blob/master/doc/api.md)）

## **安装**
```
npm install font-carrier --save
```
## **开始**
引用font-carrier后，创建一个空白字体，或者解析一个已有的字体，都可以得到一个字体对象
``` js
var fontCarrier = require('font-carrier')

//创建空白字体对象
var font = fontCarrier.create()

//从其他字体解析
var transFont = fontCarrier.transfer('./test/test.ttf')
```
## **示例**
### 1. 精简中文字库
在页面开发过程中，不可避免的问题就是因为中文字体问题无法100%还原设计稿，因为不同的浏览器和移动设备的兼容性及默认字体问题，很难达到统一的浏览体验，相较于可以用字库解决的英文字体，中文字体的字库又过于庞大，如果是活动页面这种文字较少并且可提前预知的情况，我们可以使用如下方法使页面中的文字更符合预期：

``` js
/*index.js*/
...
// 创建空白字体
var font = fontCarrier.create()
// 解析本地字体文件
var transFont2 = fontCarrier.transfer('./test/test.ttf')

// 从解析完成的结果中拿到需要的字 并生成新的字体文件 font2(默认为4种格式)
var gs = transFont2.getGlyph('特殊字体')
font.setGlyph(gs)
// 导出字体
font.output({
	path:'./test/font2'
})
```
代码运行结束后，目录结构如下：

![alt](http://km.midea.com/uploads/imgs/442174a89243.png)

可以看到我们已经得到了4种主流的字体文件，接下来我们通过@font-face引用字体文件并查看在页面中显示的效果：
```html
<!-- index.html -->
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
  <style type="text/css">
    @font-face {
      font-family: 'font2';
      src: url('./font2.eot'); /* IE9*/
      src: url('./font2.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
      url('./font2.ttf') format('truetype'), /* chrome、firefox、opera、Safari, Android, iOS 4.2+*/
      url('./font2.svg#iconfont'); /* iOS 4.1- */
    }
    .font-2{
      font-family:"font2";
      font-size:48px;
    }
  </style>
</head>
<body>
  <h class="font-2">特殊字体 其他正常显示</h>
</body>
</html>
```
显示效果如下：

![alt](http://km.midea.com/uploads/imgs/e9afbecff22a.png)

### 2. 针对字导出对应的svg图片

如题，话不多说~
``` js
/*index.js*/
...
// 解析本地字体文件
var transFont2 = fontCarrier.transfer('./test/test.ttf')

// 从新的字体文件中拿出单独字 并生成svg文件 export.svg
var path = transFont2.getSvg('字')
fs.writeFileSync('./test/export.svg',path)
```

这样我们就得到了svg文件啦~~/撒花

![alt](http://km.midea.com/uploads/imgs/b883dbd409ce.png)

有了svg文件及文字路径也就可以为所欲为了~~/坏笑
```html
<path style="fill: #ffd148;stroke:#2196F3;stroke-width:4;" d="M..."></path>
```
![alt](http://km.midea.com/uploads/imgs/5cbb801abd29.png)

## **总结**

上面只是利用了font-carrier的部分功能得到的两个小示例，它还有一些其他特性，如设置某个字或unicode对应的svg形状，将字体解析成对应的svg字符串等，感兴趣的同学可以了解一下~这个是我目前接触到的比较简单的操作字体的库，大家有什么更好的可以实现类似效果的方法也可以分享出来共同进步哟~

![alt](http://km.midea.com/uploads/imgs/41255c85e814.gif)