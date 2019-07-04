---
title: 关于Function.prototype.bind
date: 2019-07-04 13:38:14
tags: [JS]
---

## this
理解bind函数的前提是了解JavaScript中**this**的含义。this的指向在函数定义阶段是无法确定的，只有在函数执行的时候才能确定。下面我们通过几个例子讨论一下：
### 例子1:
```javascript
function a(){
    var name = '小明';
    console.log(this.name);
    console.log(this);
}
 
a();
// undefined
// Window
```
<!-- more -->
显然，这里的this指向Window，因为此时的调用函数的对象是Window，代码第6行可以理解为 Window.a();。

注意：在严格模式下，全局作用域下的函数中的this不指向Window，而是undefined。
### 例子2：
```javascript
var a = {
    name:"小明",
    fn:function(){
        console.log(this.name);
    }
}
 
a.fn();
// 小明
```
其实以上连个例子说明的是同一种情况：**函数中的this指向调用这个函数的对象**。如果函数被多个对象调用呢？
### 例子3：
```javascript
var a = {
    x:10,
    b:{
        x:15,
        fn:function(){
            console.log(this.x);
        }
    }
}
 
a.b.fn();
// 15
```
虽然函数fn是通过对象a被调用的，但此时this指向的是对象b。由此，我们可以将上面对this指向的定义补充为：**函数中的this指向直接调用这个函数的对象**。还有一种比较特殊的情况：
### 例子4：
```javascript
var a = {
    x:10,
    b:{
        x:15,
        fn:function(){
            console.log(this.x);
        }
    }
}
 
var c = a.b.fn;
c();
// undefined
```
为什么这里的输出是'undefined'呢？其实是同样的道理，虽然fn函数是被对象b引用的，但当赋值给c以后，函数执行的上下文就变成了Window，此时的this也就指向了Window。

如果this所在的函数是对象的构造函数又会有怎样的表现呢？
```javascript
function Fn(){
    this.name = "小明";
}
 
var a = new Fn();
console.log(a.name);
// 小明
```
当我们通过new来初始化一个实例a时，我们就相当于复制了一份Fn函数指向了对象a，所以a自然就有了name属性。
### 当this遇见return
这次为了整理this不同指向的情况查阅了一些文章，还发现了之前没有注意到的一个问题，当this遇见return时：
```javascript
function fn() 
{ 
    this.x = 10; 
    return {}; 
}
var a = new fn; 
console.log(a.x);
// undefined
 
function fn2() 
{ 
    this.x = 10; 
    return function(){};
}
var b = new fn2; 
console.log(b.x);
// undefined
```
为什么以上的输出都是undefined呢？当return的值不是对象的情况呢？
```javascript
function fn() 
{ 
    this.x = 10;
    return 1; 
}
 
var a = new fn; 
console.log(a.x);
// 10
 
function fn2() 
{ 
    this.x = 10;
    return undefined;
}
 
var b = new fn2; 
console.log(b.x);
// 10
```
由此可以得出，**如果返回值是一个对象，那么this指向的就是这个返回值，否则指向函数的实例**。

为了验证这个结果我们可以用下面的代码：
```javascript
function fn() 
{ 
    this.x = 10; 
    return {
        x:15
    }; 
}
 
var a = new fn; 
console.log(a.x);
// 15
```
需要注意的一点是虽然null是对象，但由于其特殊性，**如果返回值是null，this仍然指向函数的实例**。

到此我们基本理解了JavaScript中的this，下面我们进入重点，使用bind/apply/call如何改变this的指向。
## bind
### [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)中的语法
![](/images/1.png)
### 示例
**调用绑定函数时 thisArg 作为this参数传递给目标函数的值**
我们来创建一个最简单的绑定函数：
```javascript
this.x = 9;
var module = {
  x: 81,
  getX: function() {
    console.log(this.x)
  }
};
 
var obj = {
  x: 18
}
 
module.getX();
// 81
 
var retrieveX = module.getX;
retrieveX();  
// 9
 
var boundGetX = retrieveX.bind(obj);
boundGetX();
// 18
```
关于对getX函数的三次调用：
 * 第一次是直接通过module对象的调用，this指向module
 * 第二次是将函数复制后直接调用，this指向Window
 * 第三次是创建一个新函数，将this绑定到obj上
这是使用bind函数改变this指向的最基本的用法。

**使一个函数拥有预设的初始参数：**
当绑定函数被调用时，arg1, arg2, ...会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在它们后面。
```javascript
function list() {
  return Array.prototype.slice.call(arguments);
}
 
function addArguments(arg1, arg2) {
    return arg1 + arg2
}
 
var list1 = list(1, 2, 3); // [1, 2, 3]
 
var result1 = addArguments(1, 2); // 3
 
// 创建一个函数，它拥有预设参数列表。
var leadingThirtysevenList = list.bind(null, 37);
 
// 创建一个函数，它拥有预设的第一个参数
var addThirtySeven = addArguments.bind(null, 37);
 
var list2 = leadingThirtysevenList(); // [37]
 
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
 
var result2 = addThirtySeven(5); // 37 + 5 = 42
 
var result3 = addThirtySeven(5, 10); // 37 + 5 = 42 ，第二个参数被忽略
```

**配合setTimeout：**
在默认情况下，使用 window.setTimeout() 时，this 关键字会指向 window 对象。
```javascript
function LateBloomer() {
   this.petalCount = Math.ceil(Math.random() * 12) + 1;
 }
  
 LateBloomer.prototype.bloom = function() {
   window.setTimeout(this.declare, 1000);
 };
  
 LateBloomer.prototype.declare = function() {
   console.log('I am a beautiful flower with ' +
     this.petalCount + ' petals!');
 };
  
 var flower = new LateBloomer();
 flower.bloom(); 
 // 一秒钟后输出‘I am a beautiful flower with undefined petals!’
```
因为 this 指向 Window，所以 this.petalCount 是undefined。当类的方法中需要 this 指向类的实例时，你可能需要显式地把 this 绑定到回调函数，就不会丢失该实例的引用。
```javascript
window.setTimeout(this.declare.bind(this), 1000);
```
这里插一句与bind函数无关的话，使用es6中的[箭头函数](http://es6.ruanyifeng.com/#docs/function#%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0)也可以实现上面的效果，箭头函数可以让setTimeout里面的this，绑定定义时所在的作用域，而不是指向运行时所在的作用域。
```javascript
window.setTimeout(() => this.declare(), 1000);
```

**作为构造函数使用的绑定函数：**
<font color="red">这部分演示了 JavaScript 的能力并且记录了 bind() 的超前用法。以下展示的方法并不是最佳的解决方案，且可能不应该用在任何生产环境中。</font>
语法介绍中有提到，如果使用 new 运算符构造绑定函数，则忽略 thisArg 。不过提供的参数列表仍然会插入到构造函数调用时的参数列表之前。
```javascript
function Point(x, y) {
  this.x = x;
  this.y = y;
}
 
Point.prototype.toString = function() {
  return this.x + ',' + this.y;
};
 
var obj = {x: 1, y: 2};
var YAxisPoint = Point.bind(obj,0);
 
var axisPoint = new YAxisPoint(5);
console.log(axisPoint.toString())
// '0,5'
 
 
YAxisPoint(13);
console.log(obj);
// {x: 0, y: 13}
```
## 与[apply](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)/[call](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)的区别
bind()、apply()与call()都可以改变函数中this的指向，他们之间又有什么区别呢？我们先来看一下 apply() 与 call() 的使用情形：
```javascript
var a = {
    name:'小明',
    fn:function(){
        console.log(this.name);
    }
}
var b = a.fn;
b.apply(a);
// 小明
 
 
var c = a.fn;
c.call(a);
// 小明
```
通过apply和call方法，函数中的this都指向了参数对象a，看起来好像没什么区别，但当函数有多个参数的时候呢？
```javascript
var a = {
    name:'小明',
    fn:function(age, work){
        console.log(this.name + '今年' + age + '岁，工作是' + work);
    }
}
var b = a.fn;
b.apply(a,[24, '程序员']);
// 小明今年24岁，工作是程序员
 
 
var c = a.fn;
c.call(a, 24, '程序员');
// 小明今年24岁，工作是程序员
```
可以看到，apply和call的区别在于参数的格式，apply的第二个参数必须是数组。需要注意的是如果apply和call的第一个参数写的是null，那么this指向的是window对象：
```javascript
var a = {
    name:'小明',
    fn:function(){
        console.log(this);
    }
}
 
var b = a.fn;
b.apply(null);
// Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, parent: Window, …}
```
看到这里大家应该发现了，bind与apply/call的区别在于bind不会立即执行，只会返回一个修改过的函数，而apply和call会立即执行。bind传参的格式与call方法是相同的，可以接受多个参数。

**总结**
bind/apply与call的相同点是都可以改变函数中this的指向。区别在于，apply和call都是改变this的指向并立即执行这个函数但传参格式不同，bind方法可以让对应的函数想什么时候调就什么时候调用，并且可以在执行的时候传递参数。大家可以根据不同的需求选择对应的函数。
