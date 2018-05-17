---
title: 一个简单的图形转换svg动画
date: 2018-03-20 11:36:02
tags: [SVG, 动画]
---

每次看到有趣的svg动画都会抑制不住的右击查看源码，然后一如既往的被复杂的svg形状标签，绘图指令和不断变换的坐标参数搞晕。其实这也没有看起来那么难，毕竟已经有很多成熟的svg动画库供我们选择了，但是在使用之前，了解基本的svg知识还是很有必要的~所以，接下来就从一个通过改变路径数据转换简单图形的动画开始吧。
<!--more-->
![](http://km.midea.com/uploads/imgs/5fdb411b040d.gif)

## 准备工作
动画中的星星和爱心可以通过svg的path元素及其对应的三次贝塞尔曲线指令绘制出来。

使用svg的path元素可以创建复杂又精致的动画，path元素的形状是通过属性d定义的，属性d的值是一个“绘图命令+参数”的序列，如下所示为一个简单的svg路径示例：

```html
<path d="M10 10 L75 10 L75 75 Z" />
```

其中的M、L、Z就是绘图命令，数字为命令的参数(即相对于用户坐标系统的坐标)。每一个命令都有两种表示方法，一种是大写字母，表示采用绝对定位；另一种是小写字母，表示采用相对定位。

画一颗萌萌的小星星呢，我们就要用到C命令了。C对应的是三次贝塞尔曲线，说到这就不可避免的要说一下贝塞尔曲线了。

相信我们大家都接触过贝塞尔曲线，它在canvas、css动画以及svg中都扮演着很重要的角色，包括线性贝塞尔曲线、二次贝塞尔曲线、三次贝塞尔曲线、四次贝塞尔曲线…对应的数学公式呢，也是很长很长…(对数学)感兴趣的同学可以戳这里，wiki上还是有详细介绍的，不过这里我们暂时不仔细研究，因为真的是很长很长…关于贝塞尔曲线是如何绘制的，wiki上也给出了很生动的动图，这里放一张五次贝塞尔曲线的演示动画大家感受下…

![alt](http://km.midea.com/uploads/imgs/166175f57be1.gif)
<font color="#666">五次贝济埃曲线演示动画，t在[0,1]区间</font>

贝塞尔曲线的绘制呢，是基于数学公式的，属于底层一点的知识，暂不了解也不影响我们画一颗小星星，所以大家不要怕。我们需要关心的是贝塞尔曲线的指令，绘制的事情交给浏览器就好，那就说回画小星星的C指令吧~

三次贝塞尔曲线需要定义一个点和两个控制点，所以用C命令创建三次贝塞尔曲线，需要设置三组坐标参数：

```
C x1 y1, x2 y2, x y (or c dx1 dy1, dx2 dy2, dx dy)
```

这里的最后一个坐标(x,y)表示的是曲线的终点，另外两个坐标是控制点，(x1,y1)是起点的控制点，(x2,y2)是终点的控制点。控制点描述的是曲线起始点的斜率，曲线上各个点的斜率，是从起点斜率到终点斜率的渐变过程。图示如下：

![alt](http://km.midea.com/uploads/imgs/58907834e583.png)

这里是一个更好理解控制点与曲线关系的[codepen示例](https://codepen.io/thebabydino/pen/EKLNvZ)。

因为我们需要用js代码来控制路径的变化，所以直接将绘制的代码也放在js中可以更方便的控制变量，这意味着我们html部分的代码相当简单：

```html
<svg>
  <path id='shape'/>
</svg>
```

为保证svg在各种设备上的显示完全一致(主要针对不同单位)，我们给svg设置一个viewBox属性，其宽高为D，左上角坐标为`(-.5D,-.5D)`，这保证了`(0,0)`点在svg图像的正中间，更方便我们进行坐标的计算。

```javascript
const _SVG = document.querySelector('svg'),         // 获取svg元素
      _SHAPE = document.getElementById('shape'),    // 获取路径元素
      D = 1000,                                     // 设置viewBox属性
      O = { ini: {}, fin: {}, afn: {} };            // 保存形状属性值

(function init() {
  _SVG.setAttribute('viewBox', [-.5*D, -.5*D, D, D].join(' '));
})();
```

两个形状之间的曲线的端点及控制点的对应情况如下图所示：

![alt](http://km.midea.com/uploads/imgs/50231301b0d6.PNG)

[<font color="#666">戳这里查看codepen示例</font>](https://codepen.io/thebabydino/pen/NayELo)

从图中我们可以看出每个点的动画轨迹就是其坐标值的变化范围，星星的图形轮廓通过二次贝塞尔曲线指令(即每条曲线只有一个控制点)也可以实现，但为了与变化后的爱心图形中的控制点一一对应，这里我们也通过三次贝塞尔曲线指令绘制，只不过两个控制点重合。动画效果除了对应控制点，为了达到上述gif图的效果，使星星的一角朝上，我们还要加入旋转操作。整个动画的绘制可以分为四个步骤：

 1. 得到星星图案对应的点坐标
 2. 得到爱心图案对应的点坐标
 3. 将两个图案的点一一对应，确定其坐标变化范围
 4. 动起来！

## 画一个星星
![alt](http://km.midea.com/uploads/imgs/ea3f55c69d10.PNG)

如上图所示，我们将正五角星五条曲线的交点作为端点，五个角的顶点作为控制点。
```html
<svg>
  <path id='shape'/>
</svg>
<script type="text/javascript">
  const _SVG = document.querySelector('svg'), 
        _SHAPE = document.getElementById('shape'), 
		D = 1000,   // viewBox尺寸
		O = {},     // data object
		P = 5;      // 多边形顶点数
  
  function getStarPoints(f = .5) {
    const RCO = f*D,                    // 大五边形外接圆半径
	      BAS = 2*(2*Math.PI/P),        // 星星的基本角度
		  BAC = 2*Math.PI/P,            // 大五边形的基本角度
		  RI = RCO*Math.cos(.5*BAS),    // 小五边形内切圆半径 
		  RCI = RI/Math.cos(.5*BAC),    // 小五边形外接圆半径
		  ND = 2*P,                     // 控制点与端点的总数
		  BAD = 2*Math.PI/ND,           // 所有点分布的基本角度
		  PTS = [];                     // 保存所有点坐标

	for(let i = 0; i < ND; i++) {
	  let cr = i%2 ? RCI : RCO,         // 判断是端点还是控制点
	  ca = i*BAD,                       // 当前点与0°的夹角
      x = Math.round(cr*Math.cos(ca)),  // 横坐标
	  y = Math.round(cr*Math.sin(ca));  // 纵坐标
	  PTS.push([x, y]);
	  if(!(i%2)) PTS.push([x, y]);      // 重复保存控制点
	}
	return PTS
  };

  (function init() {	
	_SVG.setAttribute('viewBox', [-.5*D, -.5*D, D, D].join(' '));
	O.d = {
	  ini: getStarPoints(), 
	  afn: function(pts) {
	    return pts.reduce((a, c, i) => {
	      return a + (i%3 ? ' ' : 'C') + c
	    }, `M${pts[pts.length - 1]}`)
	  }
    };
    for(let p in O) _SHAPE.setAttribute(p, O[p].afn(O[p].ini))
  })();
</script>
```
上述代码得到的星星图案如下所示：

![](http://km.midea.com/uploads/imgs/a842f1bfc165.PNG)

可以看出星星的形状已经绘制完成了，但角度与我们预期的不同，我们希望星星的第一个角是指向下方的，然而上述代码得到的第一个角是指向右方。这是因为绘图时我们默认是从0°开始的，所以只需在上述代码的第23行进行角度计算时加上90°就可以了。

```javascript
ca = i*BAD + .5*Math.PI
```

接下来我们设置星星的初始旋转角度为-180°，填充颜色为黄色，并封装一个设置属性的函数，如下：

```javascript
function fnStr(fname, farg) { return `${fname}(${farg})` };

(function init() {
  ...
  O.fill = { ini: [255, 215, 0],  afn: (rgb) => fnStr('rgb', rgb) };
  O.transform = { ini: -180,  afn: (ang) => fnStr('rotate', ang) };
  ...
})();
```

这时候我们就得到一个萌萌哒胖星星啦~

![alt](http://km.midea.com/uploads/imgs/e3b782ef39de.PNG)

[<font color="#666">戳这里查看codepen示例</font>](https://codepen.io/thebabydino/pen/wrRWJN)
## 画一个爱心
![alt](http://km.midea.com/uploads/imgs/fc0d95843bf3.PNG)
<font color="#666">图中的D0/D1/E0/E1并非实际上的控制点，四段曲线为1/4圆，计算时会取近似的三次曲线的控制点</font>
对应的端点及控制点坐标如下：
```javascript
function getHeartPoints(f = .25) {
  const R = f*D,                        // 辅助圆半径
        RC = Math.round(R/Math.SQRT2),  // 辅助圆内接正方形边长
        XT = 0, YT = -RC,               // T(x,y)
        XA = 2*RC, YA = -RC,            // A0(x,y)
        XB = 2*RC, YB = RC,             // B0(x,y)
        XC = 0, YC = 3*RC,              // C(x,y)
        XD = RC, YD = -2*RC,            // D0(x,y)
        XE = 3*RC, YE = 0,              // E0(x,y)
        
        C = .551915,                    // 四分之一圆的三次曲线常量近似值
        CC = 1 - C, 
        // TD间的控制点坐标
        XTD = Math.round(CC*XT + C*XD), YTD = Math.round(CC*YT + C*YD), 
        // AD间的控制点坐标
        XAD = Math.round(CC*XA + C*XD), YAD = Math.round(CC*YA + C*YD), 
        // AE间的控制点坐标
        XAE = Math.round(CC*XA + C*XE), YAE = Math.round(CC*YA + C*YE), 
        // BE间的控制点坐标
        XBE = Math.round(CC*XB + C*XE), YBE = Math.round(CC*YB + C*YE);
        
  return [
    [XC, YC], [XC, YC], [-XB, YB], 
    [-XBE, YBE], [-XAE, YAE], [-XA, YA], 
    [-XAD, YAD], [-XTD, YTD], [XT, YT], 
    [XTD, YTD], [XAD, YAD], [XA, YA], 
    [XAE, YAE], [XBE, YBE], [XB, YB]
  ];
};
```
与填充星星颜色和赋初始旋转值相似，如下所示，在init方法中加入对应的爱心形状的参数：
```javascript
O.d = {
  ini: getStarPoints(), 
  fin: getHeartPoints(), 
  afn: function(pts) {
    return pts.reduce((a, c, i) => {
        return a + (i%3 ? ' ' : 'C') + c
    }, `M${pts[pts.length - 1]}`)
  }
};

O.transform = {
  ini: -180, 
  fin: 0, 
  afn: (ang) => fnStr('rotate', ang)
};

O.fill = {
  ini: [255, 215, 0], 
  fin: [220, 20, 60], 
  afn: (rgb) => fnStr('rgb', rgb)
};

 for(let p in O) _SHAPE.setAttribute(p, O[p].afn(O[p].fin))
```
这样我们就得到了一颗如下图所示的红色爱心啦~

![alt](http://km.midea.com/uploads/imgs/ea3936ab0cff.PNG)

## 在两种形状间切换

在click事件发生时，我们希望svg切换为另一种形状。解决方案是，用一个值为1或-1的`dir`变量来标识两种不同的形状，1对应为心形，-1为星星，每次`click`事件发生时，更改`dir`的值，再根据`dir`的正负`setAttribute`。代码如下：

```javascript
let dir = -1;

(function init() {	
  /* same as before */
	
  _SHAPE.addEventListener('click', e => {
    dir *= -1;
		
    for(let p in O)
      _SHAPE.setAttribute(p, O[p].afn(O[p][dir > 0 ? 'fin' : 'ini']));
  }, false);
})();
```

[<font color="#666">戳这里查看codepen示例</font>](https://codepen.io/thebabydino/pen/wPwKMw)

这种变化显然不是我们需要的结果，我们想要的是一个渐变的过程。接下来就是这个转换动画的重点，我们可以自己创建一个逐帧动画。首先确定动画的帧数并定义可能用到的时间函数：
```javascript
const NF = 50, 
      TFN = {
        'ease-out': function(k) {
          return 1 - Math.pow(1 - k, 1.675)
        }, 
        'ease-in-out': function(k) {
          return .5*(Math.sin((k - .5)*Math.PI) + 1)
        },
        'bounce-ini-fin': function(k, s = -.65*Math.PI, e = -s) {
          return (Math.sin(k*(e - s) + s) - Math.sin(s))/(Math.sin(e) - Math.sin(s))
        }
      };
```

然后，我们给两个形状不同的转换属性分别赋予不同的转换时间函数：

```javascript
(function init() {	
  /* same as before */
	
  O.d = {
    /* same as before */
    tfn: 'ease-in-out'
  };
	
  O.transform = {
    /* same as before */
    tfn: 'bounce-ini-fin'
  };
  	
  O.fill = {
    /* same as before */
    tfn: 'ease-out'
  };

  /* same as before */
})();
```

接下来声明一个rID，来表示当前执行的函数，cf变量用于表示当前帧。在单击动作发生时我们开始调用`update`函数，然后在每次刷新时不断回调`update`，直到转换完成，调用`stopAni`函数来退出这个动画循环。在`update`函数中，我们更新当前帧`cf`，计算一个进度`k`，用来判断循环是否结束。代码如下：

```javascript
let rID = null, cf = 0, m;

function stopAni() {
  cancelAnimationFrame(rID);
  rID = null;  
};

function update() {
  cf += dir;
	
  let k = cf/NF;
  
  if(!(cf%NF)) {
    stopAni();
    return
  }
  
  rID = requestAnimationFrame(update)
};
```
上述代码中声明的变量`m`用于当形状从结束状态(心形)回到初始状态(星星)时保证时间函数的入参始终大于0。这时，我们就可以将`click`事件更新为如下：

```javascript
addEventListener('click', e => {
  if(rID) stopAni();
  dir *= -1;
  m = .5*(1 - dir);
  update();
}, false);
```

通过`update`函数，我们想要根据计算得到的进度`k`来设置在`update`过程中不断变化的属性，所以我们创建这样一个函数，用来计算数字（包括数组中的数字）之间的范围。

```javascript
function range(ini, fin) {
  return typeof ini == 'number' ? 
         fin - ini : 
         ini.map((c, i) => range(ini[i], fin[i]))
};

(function init() {	
  /* same as before */
	
  for(let p in O) {
    O[p].rng = range(O[p].ini, O[p].fin);
    _SHAPE.setAttribute(p, O[p].afn(O[p].ini));
  }
	
  /* same as before */
})();
```

接下来定义一个计算各个属性中间值的函数`int`(interpolation插值函数)，这个函数的入参分别为当前属性的初始值(ini)，数值变化范围(rng)，时间函数(tfn)和进度(k)：

```javascript
function int(ini, rng, tfn, k) {
  return typeof ini == 'number' ? 
         Math.round(ini + (m + dir*tfn(m + dir*k))*rng) : 
         ini.map((c, i) => int(ini[i], rng[i], tfn, k))
};
```

接下来在`update`函数中加入获取当前属性值并更新属性值的代码：

```javascript
function update() {	
  /* same as before */
	
  for(let p in O) {
    let c = O[p];

    _SHAPE.setAttribute(p, c.afn(int(c.ini, c.rng, TFN[c.tfn], k)));
  }
	
  /* same as before */
};
```

这时我们就可以得到一个可以相互转换图形的动画啦~

![alt](http://km.midea.com/uploads/imgs/cb815bce955f.gif)

[<font color="#666">戳这里查看codepen示例</font>](https://codepen.io/thebabydino/pen/LOPpBM)

可以看出，这与我们想要实现的动画效果就只差第二次点击时的旋转方向了，我们需要在第二次点击时心形继续沿第一次点击时形状的旋转方向旋转并转化回起始的星星图形，这里我们为`transform`属性引入一个标识连续的值并相对应的调整`update`和`int`函数：

```javascript
function int(ini, rng, tfn, k, cnt) {
  return typeof ini == 'number' ? 
         Math.round(ini + cnt*(m + dir*tfn(m + dir*k))*rng) : 
         ini.map((c, i) => int(ini[i], rng[i], tfn, k, cnt))
};

function update() {	
  /* same as before */
	
  for(let p in O) {
    let c = O[p];

    _SHAPE.setAttribute(p, c.afn(int(c.ini, c.rng, TFN[c.tfn], k, c.cnt ? dir : 1)));
  }
	
  /* same as before */
};

(function init() {	
  /* same as before */
	
  O.transform = {
    ini: -180, 
    fin: 0, 
    afn: (ang) => fnStr('rotate', ang),
    tfn: 'bounce-ini-fin',
    cnt: 1
  };
	
  /* same as before */
})();
```
现在我们就可以得到我们预期的小动画啦~

![alt](http://km.midea.com/uploads/imgs/6824561ef997.gif)

[<font color="#666">戳这里查看codepen示例</font>](https://codepen.io/thebabydino/pen/LpqEmJ)

> 参考文献：
> [Creating a Star to Heart Animation with SVG and Vanilla JavaScript](https://css-tricks.com/creating-star-heart-animation-svg-vanilla-javascript/)
> [Bézier curve - Wikipedia](https://en.wikipedia.org/wiki/B%C3%A9zier_curve)
> [Paths - SVG | MDN](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths)