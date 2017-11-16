---
title: 《JavaScript设计模式与开发实践》——this、call和apply
date: 2017-05-19 03:47:45
tags: [设计模式, JS, 读书笔记]
---
# 第2章 this、call和apply

本章主要介绍了this关键字，Function.prototype.call和Function.prototype.apply这两个方法，是学习多种设计模式前应准确掌握的概念。

<!-- more -->

## this

JavaScript中的this总是指向一个对象，这个对象是在运行时**基于函数的执行环境动态绑定的**，与函数声明时的环境无关。

this的指向大致分为以下4种：
* 作为对象的方法调用
	当函数作为对象的方法被调用时，this指向该对象：
``` javascript
var obj = {
	a: 1,
	getA: function() {
		alert(this === obj);	// true
		alert(this.a);		// 1
	}
}

obj.getA();
```
* 作为普通函数调用
	this指向全局对象，在JavaScript中，这个全局对象是window对象。
``` javascript
window.name = 'globalName';

var myObject = {
	name: 'sven',
	getName: function() {
		return this.name;
	}
}

var getName = myObject.getName;
console.log(getName());		//globalName
```
* 构造器调用
	JavaScript可以从构造器中创建对象，大部分函数都可以当作构造器使用，构造器与普通函数的区别在于调用的方式。当用new运算符调用函数时，改函数总会返回一个对象，通常情况下，构造器里的this就指向返回的这个对象：
``` javascript
var MyClass = function() {
	this.name = 'sven';
};

var obj = new MyClass();
console.log(obj.name);		//sven
```
	特殊情况为构造器显式的返回了一个对象，则运算结果会返回这个对象：
``` javascript
var MyClass = function() {
	this.name = 'sven';
	return {		//显示的返回一个对象
		name: 'anne'
	}
};

var obj = new MyClass();
console.log(obj.name);		//anne
```
	则最后的输出结果为'anne'。
	注：只有在返回的是一个对象时才会发生上述情况，若是返回了其他的非对象的数据类型，则与不显式的返回数据结果一样。
* Function.prototype.call或Function.prototype.apply的调用
	和其他函数相比，Function.prototype.call或Function.prototype.apply可以动态的改变传入函数的this:
``` javascript
var obj1 = {
	name: 'sven',
	getName: function() {
		return this.name;
	}
}

var obj2 = {
	name: 'anne'
}

console.log(obj1.getName());			//sven
console.log(obj1.getName.call(obj2);		//anne
```

## call和apply

### call和apply的区别

两个方法的区别在于传入参数的形式不同。
apply接受两个参数，第一个参数指定了函数体内的this对象的指向，第二个参数为一个带下标的集合（可以是数组，也可以是类数组），apply把这个集合中的元素作为参数传入被调用的函数。
call传入的参数数量不固定，第一个参数与apply接受的第一个参数相同，从第二个参数开始往后，每个参数被依次传入函数。
``` javascript
function func(a, b, c) {
	alert([a, b, c]);		// [a, b, c]
}

func.apply(null, [1, 2, 3]);
func.call(null, 1, 2, 3);
```
在使用apply或call的时候，如果传入的第一个参数为null，this会指向默认的宿主函数，在浏览器中则为window，但在严格模式下this指向null。
当我们使用这两种方法的目的不在于改变this的指向而是有其他目的时，比如借用其他对象的方法，则可以传入null来代替某个具体对象。

### call和apply的用途
* 改变this指向
* Function.prototype.bind
	利用apply和call来模拟bind方法：
* 借用其他对象的方法