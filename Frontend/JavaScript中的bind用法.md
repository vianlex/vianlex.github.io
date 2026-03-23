# 函数对象 bind、call、apply 方法的用法

函数对象的 call、apply、bind 方法，作用是改变函数执行时的上下文，简单来说就是改变函数运行时的 this 指向。这三个方法的区别如下：

- bind 方法返回一个新的函数对象，并将新函数运行时 this 绑定到指定的对象（永久绑定），apply、call 方法的作用是立即执行当前函数并改变函数运行时 this 指向（临时改变）。 
- 三个方法，都可以传递函数的参数，apply 是通过数组方式传递，bind 和 call 是通过参数列表方式传递，注意 bind 可以分多次传递。 


## bind 用法

```JavaScript

var a = 1
function func(){
    console.log("this.a = " + this.a)
}

/** func 未指定调用者，故 this 默认指向 window 故输出以下结果  */
func() // 输出 this.a = 1 

var obj = {a: 2}
/**
 *  func 函数对象调用 bind 方法，返回一个新的函数，并且该新函数运行时 this 指向 obj
 */
var bindFunc = func.bind(obj)
/** bindFunc 函数运行时 this 指向 obj 所以 this.a == obj.a */
bindFunc() // 输出 this.a = 2



function func1(b, c){
    console.log("this.a = " + this.a)
    console.log("b = " + b)
    console.log("c = " + c)
}
// func1 函数对象调用 bind 方法绑定函数运行 this 指向，如果指定函数的部分参数，则传递的部分参数为固定值，新函数只需要传递剩余的部分参数
bindFunc1 = func1.bind({a: 3}, 4)
// 输出 this.a = 2，b = 4, c = 6
bindFunc1(5, 6)

// 当指向全部参数时，则新函数不需要在传递参数，如果传递新的参数将会被丢弃
bindFunc1 = func1.bind({a:2}, 2, 2)
// 输出 this.a = 2, b = 2, c = c 
bindFunc1()
// 输出 this.a = 2, b = 2, c = c 
bindFunc1(3,3)


// 求最大值
function max() {
    if(arguments.length == 0){
        return 
    }
    let maxValue = arguments[0]
    for(index = 1;index < arguments.length;index++){
        if(arguments[index] > maxValue){
            maxValue = arguments[index]
        }
    }
    return maxValue
}

// 输出 5
max(1,2,3,4,5)
// 通过 bind 固定部分参数
var iMax =  max.bind({}, 10,11)
// 输出 11
console.log(iMax(1,2,3,4,5))

// 通过 bind 实现部分参数，固定传值
function func3(...arg){
    /**
     *  arguments 是所有非箭头函数中都有的局部变量，是一个数组，存放的是函数参数
     *  数组的下标，即参数的位置
     * */ 
    console.log("第一个参数的值：" + arguments[0])
    console.log(arg)
}
/**
 * 输出函数的第一个参数值为 1
 * arg 是可变参数，可变参数是一个数组，故输出 [1, 2]
 */
func3(1, 2)
// 通过 bind 方法，固定设置部分参数
var bindFunc3 = func3.bind({}, 1,2)
/**
 * 函数第一个参数值输出为 1
 * arg 数组输出值为 [1, 2, 4, 5]
 */
bindFunc3(4,5)

```

### bind 方法的简单原理实现

```JavaScript

function myBind(bindObj){
   // 判断调用对象是否为函数
    if (typeof this !== "function") {
        throw new TypeError("Error");
    }
    // 获取调用 myBind 方法的函数
    fn = this
    if(!bindObj){
        // 利用闭包
        return function(){
            return fn(...arguments)
        }
    }
    let fixArgs = [...arguments].slice(1)
    // 利用闭包
    return function (){
        return fn.apply(bindObj, fixArgs.concat(...arguments))
    }
}


function func(b){
    console.log("this.a = " + this.a)
    console.log("b = " + b)
}

// 直接绑定到 func 函数对象上
func.myBind = myBind

var func1 = func.myBind({a: 20})
// 输出 this.a = 20 , b = 30
func1(30)
func1 = func.myBind({a: 20}, 20)
// 输出 this.a = 20 , b = 20
func1()

// 绑定到 Function 构造函数的 prototype 属性上
Function.prototype.myBind = myBind
// 字面量函数对象，默认的构造函数是 Function
function func2(b){
    console.log("this.a = " + this.a)
    console.log("b = " + b)
}
// 输出 this.a = 30 , b = 20
func2.myBind({a: 30}, 20)


```
 
## apply 的用法

apply 方法改变函数运行时 this 的指向，并且传递参数时，必须是以数组的方式传递，如下例子：

```JavaScript

function func(b){
    console.log("this.a  = " + this.a )
    console.log("b = " + b)
}
// 输出 this.a = 10, b = 60
func.apply({a: 10}, [60])
// 输出 this.a = 10, b = undefined
func.apply({a: 10})

```

## call 的用法

call 用法跟 apply 的用法一样，只是传递函数参数的方式不一样，call 是以参数列表的方式传递函数参数

```JavaScript

function func(b){
    console.log("this.a  = " + this.a )
    console.log("b = " + b)
}
// 输出 this.a = 10, b = 60
func.call({a: 10}, 60)
// 输出 this.a = 10, b = undefined
func.call({a: 10})

```
