# JavaScript 事件循环机制 

## JavaScript 是单线程

JavaScript 是单线程的，语言的设计起初也是不支持创建多线程的，JavaScript 主要用于用户交互，以及  DOM 和 CSSOM 操作, 如果设计多成线程，会涉及共享资源问题，增加程序的复杂性而引入不必要的问题。

为了利用多核 CPU 的处理复杂的计算任务，HTML5 提出 Web Worker 标准，允许 JavaScript 创建多个线程，但是子工作线程完全受主线程控制，工作线程运行在另一个上下文中，无法使用 Window 对象，并且不能操作 DOM，因此工作线程并没有改变 JavaScript 单线程的本质。


## 事件循环

JavaScript 是单线程的，所有同步的代码都是在主线程的执行栈上运行的，并且 JavaScript 运行时，还会创建一个或者多个待处理消息（回调函数）的任务队列，当主线程任务执行完成后，JavaScript 引擎会检查任务队列是否有可运行的任务，如果有，则会从任务队列中读取任务执行，因为只有一个主线程，所以只有前一个读取的任务执行完成并且主线程没有其他任务可执行，才会继续检查任务队列是否有准备好可运行的任务的，有则读取执行，如此反复循环的检查读取准备好可运行的任务，则称为事件循环。具体可见 Philip Roberts 演讲的演示，如下截图：

![JavaScript代码运行可视化图](./images/JavaScript代码运行可视化图.png)


事件循环的工作原理：

1. JavaScript 代码在主线程执行栈中运行，当执行同步代码时，代码会按照调用顺序进入执行栈，执行后完成出栈，当执行异步代码（如 setTimeout、setInterval、DOM 事件、XMLHttpRequest 等）时，JavaScript 引擎会将异步任务交给 Tab 页签进程的其他线程去处理（其他线程处理结束后，会将回调消息放入任务队列中），然后继续执行同步代码。

2. 当所有同步代码执行完成后，JavaScript 引擎会从任务队列中读取可运行的消息任务，然后在主线程的执行栈中执行，当执行栈的代码执行完后，又继续从任务队列中读取可运行的消息任务，... 如此循环往复。



## 微任务队列

DOM 规范中规定的 MutationObserver 和 ECMA 规范中规定的 Promise 都是异步的 API，当 JavaScript 引擎按顺序执行同步代码时，如果遇到它们，JavaScript 引擎会将它们交由 Tab 页签进程的其他线程执行处理，JavaScript 运行时除了创建前面说到的任务队列（称宏任务队列）还会创建另一个任务队列（微任务队列），主要用于存放  MutationObserver 和 Promise API 处理结束回调消息，微任务队列比任务队列的优先级高，每次一次主线程主栈执行完后，都会优先检查微任务队列是否有可运行的消息任务，有则读微任务队列的消息，然后在主线线程执行栈中执行，执行后有又继续检查微任务队列，当未任务队列没有可运行的消息任务时，才会去检查任务队列，如果有可运行的任务，则读取到主线程栈中执行，执行栈执行完后，又继续检查微任务队列，就这样一直循环持续着，直到两个队列都为空。


## 测试例子

```javaScript 



```







## 参考链接

1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Event_loop
2. https://html.spec.whatwg.org/multipage/webappapis.html#event-loops
3. https://latentflip.com/loupe
