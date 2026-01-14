# 变量声明和作用域

## var

- var 声明的变量属于函数作用域，在函数内声明的变量才是局部变量，非函数内声明的变量是全局作用域变量
- var 声明的变量会被提升到函数或全局作用域的顶部，注意函数内的变量只提升声明，并不提升赋值
- var 允许重复声明变量，当重复声明时没有赋值，JS 引擎会忽略后续的声明，当重复声明时赋值，JS 引擎会用新值覆盖旧值。

```js
// a 声明和赋值被提升到顶部
console.log("hello " + a) // 输出 hello 90
for(let i = 0; i< 10; i++) {
    var a = 90    
}

var xx = 120
// 忽略声明，不会忽略赋值
var xx ;
console.log(xx) // 输出 120

// 使用 ! 执行函数
!function func1() {
    // 注意 x 变量只有声明被提升到函数顶部，赋值并没有提升
    console.log("hello " + x) // 输出 hello undefined
    var x = 100
    console.log("hello" + x) // 输出 hello 100
    // 未声明的变量不能使用，声明但未赋值的变量，默认值为 undefined
    console.log(xx) // 报错提示：xx is not defined
}()


!function(a){
  console.log(a) // 输出 100
  // 形参 a 已声明过，引擎会忽略掉该声明，只会重新赋值
  var a = 200
  console.log(a) // 输出 200
  // arguments 函数的内置参数对象（存的应该是引用）
  console.log(arguments[0]) // 输出 200 并不是 100
}(100)

```

## let 

- let 声明的变量属于块级作用域（在 {} 内部声明的变量都属于局部变量）
- let 变量在声明之前访问变量会报错，称为暂时性死区
- let 变量不允许重复声明，重复声明会报错


```js

if (true) {
  console.log(x); // 报错：Cannot access 'x' before initialization
  let x = 10;
  console.log(x); // 10
}
console.log(x); // 报错：x is not defined

```

## const

- const 声明的变量属于块级作用域（在 {} 内部声明的变量都属于局部变量）
- const 声明的变量不允许重复和重复赋值

```js

const PI = 3.14;
PI = 3.14159; // 报错：Assignment to constant variable

```

## 词法作用域

词法作用域（Lexical Scope），也称为静态作用域，是指变量的作用域在代码编写时就已经确定，而不是在代码运行时确定。换句话说，词法作用域是由代码的结构和位置决定的，与函数的调用方式无关。

### 词法作用域的特点

- 静态性：作用域在代码编写时就已经确定，不会因为函数的调用方式而改变
- 嵌套性：内部作用域可以访问外部作用域的变量，但外部作用域不能访问内部作用域的变量
- 作用域链：当访问一个变量时，JavaScript 引擎会沿着词法作用域链向上查找

### 词法作用域 vs 动态作用域

JavaScript 采用的是词法作用域，而不是动态作用域。

- 词法作用域：作用域在代码编写时确定，与函数定义的位置有关
- 动态作用域：作用域在代码运行时确定，与函数调用的位置有关

```js

// 闭包例子
function createCounter() {
  let count = 0;
  return function() {
    count++;
    console.log(count);
  };
}
const counter = createCounter();
counter(); // 1

```


## 作用域链

当访问一个变量时，JavaScript 引擎会从当前作用域开始查找，如果找不到，会逐级向上查找，直到全局作用域。这种链式查找的过程就是作用域链。
















