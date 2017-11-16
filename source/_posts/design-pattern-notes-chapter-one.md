---
title: 《JavaScript设计模式与开发实践》——面向对象的JavaScript
date: 2017-05-18 04:22:28
tags: [设计模式, JS, 读书笔记, 面向对象]
---
# 第1章 面向对象的JavaScript

JavaScript没有提供传统面向对象语言中的类式继承，对象与对象间的继承关系有多种实现方式。
在正式学习之前，要了解JavaScript在面向对象方面的知识。

<!-- more -->

## 动态类型语言和鸭子类型

静态类型语言与动态类型语言的区别为，静态类型语言在编译时便已确定变量类型，动态类型语言则要到程序运行时，变量被赋值后，才具有类型。
很明显JavaScript属于**动态类型语言**，其优点为*代码量少，简洁，灵活性高*。
这一切都建立在**鸭子类型**的概念上，其通俗说法为*“如果它走起路来像鸭子，叫起来也像鸭子，那么它就是鸭子。”*
鸭子类型指导我们*只关注对象行为，而不关注对象本身。*例如，如果一个对象有pop和push方法并正确实现，它就可以被当作栈来使用。


## 多态

**多态**的实际含义是*同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果。*
其背后的思想是将“做什么”和“谁去做以及怎样去做”分离开，实现过程即，把不变的部分隔离出来，可变的部分封装起来，对应的最常用的手段为**继承**。
而JavaScript因其动态类型语言的特性，一个对象可以表示多种类型的对象，其多态性是与生俱来的。
设计模式可以理解为在某种场合下对某个问题的一种解决方案，绝大多数的设计模式的实现都离不开多态性的思想。

## 封装

**封装**的目的是*隐藏信息*，广义上的封装包括*封装数据*，*封装实现*，*封装类型*，*封装变化*。
JavaScript对数据的封装是通过用函数创建作用域来实现的。[ES6中可以通过Symbol来创建私有属性](https://github.com/lukehoban/es6features#symbols)
封装实现细节使得对象之间的耦合变松散，只通过暴露的API接口通信，其他的对象或用户不关心内部实现的细节，接口不变就不会影响程序的其他功能。
封装类型是静态类型语言中的一种重要的封装方式，对JavaScript来说是没有必要的。
从设计模式的角度出发，封装更重要的层面体现在**封装变化**，通过封装变化的方式，把系统中稳定不变的部分和容易变化的部分分离开来，使得容易变化的部分在系统演变的过程中更容易替换，最大程度的保证程序的稳定性和可扩展性。当我们把程序中变化的部分封装好后，剩下的即是稳定的，可复用的。

## 原型模式和基于原型继承的JavaScript对象系统

在以类为中心的面向对象编程语言中，对象是从类中创建来的。而在原型编程的思想中，类不是必须的，对象是通过**克隆**另一个对象所得到的。
**原型模式**不单是一种设计模式，也被成为一种编程泛型，从设计模式的角度来讲，原型模式是用于*创建对象*的一种模式，不必关心对象的类型，而是直接通过克隆来创建。
原型模式的实现关键是语言本身是否提供了*clone*方法。ECMAScript5提供了[Object.create](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)方法用来克隆对象。
在不支持Object.create方法的浏览器中，则可以使用以下代码：
``` javascript
Object.create = Object.create || function (obj) {
	var F = function(){};
	F.prototype = obj;

	return new F();
}
```
原型模式的真正目的并非需要得到一个一模一样的对象，而是提供一种便捷的方式去创建某种类型的对象，克隆也只是实现这个目的的过程和方法。
JavaScript本身是基于原型的面向对象语言，从设计模式的角度来讲，原型模式的意义不大，它的对象系统就是使用原型模式搭建的，所以这里将原型模式成为原型编程范型也许更合适。
**所有的JavaScript对象都是从某个对象上克隆来的。**
**原型编程范型至少包含以下基本规则：**

* 所有的数据都是对象。
	这种说法并不标准，JavaScript中是分为基本类型（undefined/null/number/string/boolean）和对象类型的，但基本数据类型也可以通过“包装类”的方法变成对象类型数据来处理。
	JavaScript中的根对象是Object.prototype对象。可以利用ECMAScript5提供的Object.getPrototypeOf查看对象的原型：
	``` javascript
	var obj1 = new Object();
	var obj2 = {};

	console.log(Object.getPrototypeOf(obj1) === Object.prototype);		//true
	console.log(Object.getPrototypeOf(obj2) === Object.prototype);		//true
	```

* 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它。
	在JavaScript语言里，我们不需要关心克隆的细节，它是由引擎内部负责实现的。我们只需显式的调用var obj1 = new Object()或var obj2 = {}。当时用new运算符来调用函数时，Object()并不是类，而是构造器。
* 对象会记住它的原型。
	JavaScript给对象提供了一个名为\_proto\_的隐藏属性，某个对象的\_proto\_属性默认会指向它的构造器的原型对象，即\{Constructor\}.prototype。在Chrome或FireFox这些把\_proto\_暴露出来的浏览器中，我们可以通过下述方法验证：
	``` javascript
	var a = new Object();
	console.log(a._proto_ === Object.prototype);		//true
	```
* 如果对象无法响应某个请求，它会把这个请求委托给自己的原型。
	这条规则涉及到JavaScript很重要的一个特性，即**原型链**（如果A对象是从B对象克隆而来，那么B对象就是A对象的**原型**，同时C对象又是B对象的原型，它们之间就形成了一条原型链）。这条规则即规定了请求是顺着原型链向“父类”方向查找的，原型链的委托机制就是**原型继承**的本质。下面我们通过最常用的原型继承的方法，来理解。
	``` javascript
	var obj = {name: 'sven'};

	var A = function(){};
	A.prototype = obj;

	var B = function(){};
	B.prototype = new A();

	var b = new B();
	console.log(b.name);		// sven
	```
	执行这段代码时，引擎做了以下事情：
	1. 遍历对象b中的所有属性，没有找到name这个属性。
	2. 查找name属性的请求被委托给对象b的构造器的原型，它被\_proto\_记录着并指向B.prototype，而B.prototype被设置为一个通过new A()创建出来的对象。
	3. 在该对象中依然没有找到name属性，于是请求被委托给这个对象的构造器的原型A.prototype，而A.prototype被设置为对象obj。
	4. 在对象obj中找到了name属性，并返回它的值。

注： ES6带来了新的Class语法，但其背后仍是通过原型机制来创建对象。[简单代码实例](http://jurberg.github.io/blog/2014/07/12/javascript-prototype/)：
``` javascript
Class Animal {
	constructor(name) {
		this.name = name;
	},
	getName() {
		return this.name;
	}
}

Class Dog extends Animal{
	constructor(name) {
		super(name);
	}
	speak() {
		return "woof";
	}
}

var dog = new Dog("Scamp");
console.log(dog.getName() + ' says ' + dog.speak);
```



## 小结

本书第1章前部分主要讲JavaScript面向对象的一些基本知识，有JS基础的读者可以不必花心思在上面，本文也只是对主要内容做了简短概括。

后半部分讲述到了本书的第一个设计模式——**原型模式**，它构成了JavaScript这门语言的基本。

