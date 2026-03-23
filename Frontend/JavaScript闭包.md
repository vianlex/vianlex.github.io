# JavaScript 闭包

## 闭包的理解

闭包是由函数以及声明该函数的词法环境组合而成，当内层函数引用了外层函数作用域下的变量，并且内层函数在全局环境下可访问，就形成了一个闭（Closure）作用域。即外部函数执行结束（或者内部函数不在外部函数的局部作用域内执行）时，外部函数内部定义的变量还能被内部函数引用，没有被回收。


## 闭包使用场景

### 外部函数返回内部函数

外部函数返回内部函数，内部函数在全局作用域执行时，还能还访问到已执行结束的外部函数的作用域

```JavaScript

// es6 之前 JS 只有全局作用和函数内部的局部作用域
var num = 10
function func01() {
    // 局部变量
    var num = 20 
    // func2 是内部函数，属于局部变量
    function func02() {
        console.log(num)
    }
    return func02
}

var fn = func01()
// 通过打印输出 window 全局对象查看 fn 函数的 [[Scopes]] 属性，可以查看到有一个闭包 Closure 作用域
console.log(window)
// func01 函数执行结束后，其内部定义的 num 变量，还被 fn 引用着，故输出 20
fn()



function hello(message) {
    setTimeout( function callback(){
        console.log( message );
    }, 1000 );
}
// message 是 hello 函数作用域内的变量，hello 执行结束后，message 变量还被 callback 函数引用着，故就形成了闭包
hello( "Hello World!" );


function f1() {
    var message = "i am closure variable"
    function f2() {
        console.log(message)
    }
    // f2 传递给 f3 函数执行，f2 还是能访问到 f1 中的 message 变量，可见闭包是基于词法作用域的（代码编写时变量声明的作用域）
    f3(f2)
    console.log("f1 end ")
}
function f3(fn) {
    fn()
}
// 输出 'i am closure variable' 后再输出 'f1 end'
f1()

```


### 闭包保护变量（私有变量）

```JavaScript

function hello() {
    var name = "closure"
    var age = 90
    return {
        greet() {
            // 断点可以查看到闭包作用域
            debugger
            console.log("hello " + name)
        },
        getAge() {
            age = age + 1
            return age
        }
    }
}

// name 和 age 属于函数内的局部变量，当前函数执行时，只能通过内部函数去读取或者修改，这就起到私有变量的作用
var obj = hello()
// 输出 'hello closure'
obj.greet()
// 输出 91
obj.getAge()
// 输出 92
obj.getAge()

```



## 参考链接
1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures
2. https://zhuanlan.zhihu.com/p/574913236