---
title: Node的初步了解：关于APIs,HTTP和JS ES6+
date: 2019-03-12 16:42:46
tags: [JS, 翻译, Node]
---
> 原文链接：[Get Started With Node: An Introduction To APIs, HTTP And ES6+ JavaScript](https://www.smashingmagazine.com/2019/02/node-api-http-es6-javascript/)

简要：介绍web后台应用程序的开发过程——讨论前沿的ES6+ javascript特性，超文本传输协议(HTTP)，运用APIs和JSON，使用node.js构建快速可扩展的后台。
<!-- more -->
***
你可能听说过node.js是一个“基于Chrome V8 JavaScript引擎的JavaScript运行环境”，它“使用了一个事件驱动、非阻塞式I/O的模型，使其轻量又高效”。但对一些人来说，这并不是最好的解释。
首先我们要清楚什么是Node(节点)？Node“异步”确切指的是什么？它与“同步”的区别在哪里？“事件驱动”和“非阻塞式”又是什么意思？Node如何应用在应用程序、网络和服务器的全局？
接下来我们将解答以上的所有问题，并且更系统的深入了解Node的内部工作原理、HTTP、APIs、JSON。并且利用MongoDB，Express，Lodash，Mocha和Handlebars构建我们自己的书籍管理API。

## 什么是Node.js
Node是一个在浏览器之外可以运行JavaScript的运行环境。我们可以用它来构建桌面应用程序(使用框架，如Electron)，编写web或app服务等。

## 阻塞/非阻塞以及同步/异步
假设我们要调用数据库来查询一个用户的信息，这次调用需要一定的时间，如果请求是“阻塞”的，那么意味着调用完成前，它将阻塞程序的执行。这种情况下，我们发出了一个“同步”的请求，因为它最终阻塞了线程。
所以说，同步的操作会阻塞进程或线程，直到操作完成，使线程处于“等待状态”。相对的，一个异步操作就是非阻塞式的。它允许线程继续执行，不考虑操作直至完成所花费的时间，所以线程不会进入“等待状态”。
让我们看一下另一个同步调用导致线程阻塞的示例。假设我们正在构建一个应用程序，用于比较两个天气API的结果以找出它们在温度上差异的百分比。用阻塞的方式，我们调用天气API1并等待返回结果，得到结果以后再调用天气API2并等待结果。

![同步阻塞操作的时间进程](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/e1043f01-6675-4b6a-82df-d6cbf06667ac/node-mongose-express-syncblock.png)
<font color="#999">同步阻塞操作的时间进程</font>

重要的是要意识到并不是所有的同步调用的必须阻塞，如果同步操作可以在不阻塞线程或使线程进入等待状态下的被处理，它就是非阻塞的。大多数时候，同步调用会被阻塞，完成所消耗的时间取决于多种因素，比如API服务器的速度，用户网络连接的下载速度等。
对于上图中的情况，我们必须等待相当长的一段时间才能从API1中得到结果。接下来我们还要等待一段时间才能从API2中得到结果。在等待相应的过程中，用户会注意到我们的应用程序被挂起了(UI看起来是锁定状态)，对用户体验来说非常不好。
在非阻塞的情况下，我们会得到这样的结果：
![异步非阻塞操作的时间进程](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/70a4c54a-8666-4961-bb39-8939b8c26a86/node-mongose-express-asyncnonblock.png)
<font color="#999">异步非阻塞操作的时间进程</font>

你可以清晰的看出我们执行的结果快了多少。我们同时等待两个API完成比顺序等待的结果快了将近50%。我们开始调用API1并等待其相应的同时我们就调用了API2并等待其相应。
到这里，在进入更具体的示例之前，很重要的一点是为了更简单，同步(Synchronous) 通常简写为“Sync”,异步(Asynchronous)通常简写为“Async”。你会经常在方法或函数中见到它们。


## 翻译中...
