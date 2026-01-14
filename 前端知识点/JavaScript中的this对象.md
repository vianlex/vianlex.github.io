## this 对象说明

### 执行上下文（execution context）

执行上下文（execution context）是 JavaScript 程序运行时创建的执行栈，可以理解为代码运行时都是执行上下文中执行的。
执行上下文分为以下三种：

1. 全局执行上下文（Global execution context） 

    非函数内部的代码执行都在全局上下文中运行，this 指向的是全局作用域的顶端对象（浏览器端是 Window，node 中是 global），
    JS 是单线程程序的，所以一个程序都只有一个全局执行上下文。

2. 函数执行上下文（Function execution context）
    
    当函数被调用时，会创建一个新的执行上下文即函数执行上下文，函数运行时 this 指向的是调用函数的对象，如果调用函数
    的对象是全局对象，则为 window 对象或者 undefined（严格模式下） 。

3. Eval 函数执行上下文（Eval exection context）  

    使用 eval 函数执行代码时，它也会创建一个新的执行上下文。


### 执行栈（execution context stack）

执行栈，是一种拥有 LIFO（后进先出）数据结构的栈，被用来存储代码运行时创建的所有执行上下文。

Js 是单线程程序，当 Js 引擎执行脚本代码时，会创建一个全局执行上下文并压入当前的执行栈中，遇到函数时会创建函数执行上下文栈并压入执行栈顶，
并执行该函数，当该函数执行完时会从栈顶弹出，直到所有代码执行完后，Js 引擎才将当前栈中的全局执行上下文移除。


### 全局执行上下文

一个程序执行只有一个全局执行上下文，在全局执行上下文中 this 指向的是局作用域的顶端对象（浏览器端是 Window，node 中是 global）。

```js

// 浏览器中，输出 window 对象
console.log(this)

// var 关键字声明定义的变量，默认会绑定的到全局作用域的 window 对象中
var greet01 = 'Hello World'
// let, const 关键字声明定义的变量，默认不会绑定到全局作用域（Global）中，会绑定到 Local 或者 Script 作用域中，具体看执行环境。 
let greet02 = 'Hello World'

// 输出  'Hello World' 字符串
console.log(this.greet01)
// 输出 undefined, this 指向的是全局作用域的 window 对象，故会输出 undefined
console.log(this.greet02)

// 验证上面的列子，可以通过 debugger 的方式查看变量绑定在哪个作用域（Scope）中
debugger 

// 在 node 环境中，输出 global 对象
console.log(this)

```

### 函数执行上下文

#### 普通函数中的 this 

普通函数中的 this 值，在函数被调用时才确定（运行时绑定），指向的是调用函数的对象，但是在严格模式和非严格模式之间会有一些差异，
即当函数调用对象为 window 时，在严格模式下 this 的值是 undefined。

非严格模式

```js

var greet = 'Hello World'

function hello(){
    console.log(this.greet)
}

// hello 没有指定调用对象，则 this 默认指向的 window 对象，故输出 'Hello World' 字符串
hello()

var helloObj = {
    'name' : 'HelloObj',
    sayHello(){
        console.log(this)
    }
}
// 指定 sayHello 函数的调用对象为 helloObj，故输出的 this 为 helloObj 对象
helloObj.sayHello()


var sayHello = helloObj.sayHello
// 未指定调用对象，则 sayHello 函数的默认调用对象为 window 对象, 故 this 的值为 window
sayHello()


```

严格模式

```js
// 启用严格模式
'use strict';

var greet = 'Hello World'

function hello(){
    // 'use strict'; 也可设置函数级别严格模式
    console.log(this.greet)
}

// 输出 undefined
hello()

```


#### 构造函数中的 this 

构造函数中的 this 值，指向的是构建函数创建出来的对象。

```js

function createObj(name) {
    this.name = name
    console.log("==========createObj========")
    // 输出 createObj {name: 'HelloObj'}，表示这是一个 createObj 类型的对象
    console.log(this)
}

// 通过原型链的方式绑定方法，这样每 new 一个 createObj 类型的对象，原型链上都会带有 func1 方法
createObj.prototype.func1 =  function(){
    console.log("==========func1========")
    // 输出 createObj {name: 'HelloObj'}
    console.log(this)
    console.log(this.name)
}

var helloObj = new createObj('HelloObj')

// 直接将 func2 方法绑定到 helloObj 对象，则该方法独属于 helloObj 对象的
helloObj.func2 =  function(){
    console.log("==========func2========")
    // 输出 createObj {name: 'HelloObj'}
    console.log(this)
    console.log(this.name)
}


helloObj.func1()

helloObj.func2()

```



#### 箭头函数中的 this 

箭头函数 this 的值指向闭合词法上下文的值，即箭头函数的 this 始终指向箭头函数定义时所在的对象（非运行时绑定）。

```js

var greet = 'Hello World'

let func1 = () => {
        console.log(this) 
        console.log(this.greet)
    }

// 箭头函数 func1 是在 window 对象下声明定义的, 故输出 window 对象和 'Hello World' 字符串
func1()

```


```js

var greet = 'Hello World'

var helloObj = {
    "greet" : "Hello Obj",
    /** func2 箭头函数是在 helloObj 对象创建时定义的，helloObj 创建时所在对象为 window 对象
        故其 this 指向 window 对象，如果是严格模式，则 this 指向 undefined  */
    func2 : () => {
        // 该箭头函数，this 指向 window 对象
        console.log(this)
        console.log(this.greet)
    },
    func3: function(){
        // 非箭头函数的 this 指向函数的调用对象
        console.log(this)
        console.log(this.greet)
    },
    func4: function() {
        // xx 箭头函数是在 helloObj 对象调用 func4 函数时定义的，故其 this 指向 helloObj 对象
        let xx = () =>{
            console.log(this)
            console.log(this.greet)
        }
        xx()
    }
}

// 输出 window 对象和 'Hello World' 字符串
helloObj.func2()

// 输出 helloObj 对象和 'Hello Obj' 字符串
helloObj.func3()

// 指定函数的调用对象为 helloObj, 故输出 helloObj 对象和 'Hello Obj' 字符串
helloObj.func4() 

var func = helloObj.func4
/** 函数未指定调用对象, 默认的调用对象为 window 对象,   
    则 func 函数是在 window 对象下声明定义的, 故输出 window 对象和 'Hello World' 字符串 */
func()


```








### 参考链接
1. https://tc39.es/ecma262/#sec-execution-contexts
2. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this