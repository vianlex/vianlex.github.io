# Groovy 基础知识

---

## 一、基础语法

### 1.1 变量声明

```groovy
// 动态类型
def name = "Tom"
def age = 25
def price = 9.99

// 强类型
String name = "Tom"
int age = 25
double price = 9.99

// 多重赋值
def (a, b, c) = [1, 2, 3]

// 常量
final PI = 3.14
```

### 1.2 字符串

```groovy
// 单引号（普通字符串）
def s1 = 'Hello World'

// 双引号（支持插值）
def name = "Tom"
def s2 = "Hello ${name}"       // Hello Tom
def s3 = "1 + 1 = ${1 + 1}"   // 1 + 1 = 2

// 三引号（多行）
def s4 = '''
    第一行
    第二行
    第三行
'''

// 斜杠字符串（正则友好）
def regex = /\d+\.\d+/

// 字符串操作
"hello".toUpperCase()          // HELLO
"HELLO".toLowerCase()          // hello
"hello world".split(" ")       // ["hello", "world"]
"hello".contains("ell")        // true
"hello".startsWith("he")       // true
"hello".endsWith("lo")         // true
"hello".replace("l", "r")      // herro
"  hello  ".trim()             // hello
"hello".length()               // 5
"hello"[0]                     // h
"hello"[1..3]                  // ell
```

### 1.3 数字

```groovy
def i = 100
def l = 100L
def f = 3.14f
def d = 3.14
def bi = 100G          // BigInteger
def bd = 3.14G         // BigDecimal

// 运算
10 / 3                 // 3（整除）
10 / 3.0               // 3.333...
10 % 3                 // 1
2 ** 10                // 1024（幂运算）
Math.abs(-5)           // 5
Math.max(3, 5)         // 5
Math.min(3, 5)         // 3
```

---

## 二、控制流程

### 2.1 if / else

```groovy
// 基本
if (x > 0) {
    println "正数"
} else if (x < 0) {
    println "负数"
} else {
    println "零"
}

// 三元运算符
def result = x > 0 ? "正数" : "非正数"

// Elvis 运算符（Groovy 特有）
def name = user?.name ?: "匿名"
// 等价于：user?.name != null ? user.name : "匿名"
```

### 2.2 switch

```groovy
switch (x) {
    case 1:
        println "one"
        break
    case [2, 3]:           // 匹配列表
        println "two or three"
        break
    case 4..10:            // 匹配范围
        println "4 to 10"
        break
    case String:           // 匹配类型
        println "string"
        break
    default:
        println "other"
}
```

### 2.3 循环

```groovy
// for 循环
for (int i = 0; i < 5; i++) {
    println i
}

// for-in 循环
for (item in [1, 2, 3]) {
    println item
}

// times
5.times { println it }

// upto / downto
1.upto(5) { println it }
5.downto(1) { println it }

// step
1.step(10, 2) { println it }   // 1,3,5,7,9

// while
while (x > 0) {
    x--
}

// each
[1, 2, 3].each { println it }
[1, 2, 3].eachWithIndex { item, idx -> println "$idx: $item" }
```

---

## 三、集合

### 3.1 List（列表）

```groovy
// 创建
def list = [1, 2, 3, 4, 5]
def empty = []

// 访问
list[0]                    // 1（第一个）
list[-1]                   // 5（最后一个）
list[1..3]                 // [2, 3, 4]（切片）

// 添加
list << 6                  // 追加
list.add(7)
list.addAll([8, 9])

// 删除
list.remove(0)             // 按索引
list.remove(Integer.valueOf(3))  // 按值

// 常用方法
list.size()                // 长度
list.isEmpty()             // 是否为空
list.contains(3)           // 是否包含
list.indexOf(3)            // 索引位置
list.sort()                // 排序
list.reverse()             // 反转
list.unique()              // 去重
list.flatten()             // 展平嵌套
list.join(", ")            // 连接为字符串

// 函数式操作
list.each { println it }
list.collect { it * 2 }    // map：[2,4,6,8,10]
list.findAll { it > 3 }    // filter：[4,5]
list.find { it > 3 }       // 找第一个：4
list.any { it > 4 }        // 是否有满足：true
list.every { it > 0 }      // 是否全满足：true
list.count { it > 3 }      // 计数：2
list.sum()                 // 求和：15
list.min()                 // 最小：1
list.max()                 // 最大：5
list.inject(0) { acc, v -> acc + v }  // reduce
list.groupBy { it % 2 }    // 分组：[0:[2,4], 1:[1,3,5]]
list.sort { a, b -> b <=> a }  // 自定义排序
```

### 3.2 Map（映射）

```groovy
// 创建
def map = [name: "Tom", age: 25, city: "Beijing"]
def empty = [:]

// 访问
map.name                   // Tom
map["name"]                // Tom
map.get("name", "默认")    // Tom（带默认值）

// 添加/修改
map.email = "tom@example.com"
map["phone"] = "123456"

// 删除
map.remove("city")

// 常用方法
map.size()                 // 3
map.isEmpty()              // false
map.containsKey("name")    // true
map.containsValue("Tom")   // true
map.keySet()               // [name, age, city]
map.values()               // [Tom, 25, Beijing]
map.entrySet()             // 键值对集合

// 遍历
map.each { k, v -> println "$k: $v" }
map.each { entry -> println entry }

// 函数式操作
map.collect { k, v -> "$k=$v" }   // 转换
map.findAll { k, v -> v > 20 }    // 过滤
map.find { k, v -> v == "Tom" }   // 查找
map.any { k, v -> v == "Tom" }    // 是否有
map.every { k, v -> v != null }   // 是否全满足
map.groupBy { k, v -> v.class }   // 分组
```

### 3.3 Range（范围）

```groovy
// 创建
def r1 = 1..10             // 1到10（包含）
def r2 = 1..<10            // 1到9（不含10）
def r3 = 'a'..'z'          // 字母范围

// 使用
r1.each { println it }
r1.contains(5)             // true
r1.size()                  // 10
r1.from                    // 1
r1.to                      // 10
r1.toList()                // [1,2,3,...,10]
```

### 3.4 Set（集合）

```groovy
// 创建
def set = [1, 2, 3, 2, 1] as Set   // {1, 2, 3}
def hs = new HashSet([1, 2, 3])

// 操作
set.add(4)
set.remove(1)
set.contains(2)
set.size()
```

---

## 四、闭包（Closure）

### 4.1 基本语法

```groovy
// 定义
def greet = { name -> "Hello, $name!" }

// 调用
greet("Tom")               // Hello, Tom!
greet.call("Tom")          // Hello, Tom!

// 无参数（隐式 it）
def double = { it * 2 }
double(5)                  // 10

// 多参数
def add = { a, b -> a + b }
add(3, 4)                  // 7

// 多行
def process = { x ->
    def result = x * 2
    result + 1
}
```

### 4.2 闭包特性

```groovy
// 捕获外部变量
def factor = 3
def multiply = { x -> x * factor }
multiply(5)                // 15

// 返回值（最后一行）
def calc = { x -> x * 2 }  // 自动返回 x * 2

// 显式返回
def calc2 = { x ->
    if (x > 0) return x * 2
    return 0
}

// 委托（delegate）
class Config {
    def host = "localhost"
    def port = 8080
}
def closure = { println "$host:$port" }
closure.delegate = new Config()
closure()                  // localhost:8080
```

### 4.3 常用闭包方法

```groovy
// 柯里化
def add = { a, b -> a + b }
def add5 = add.curry(5)
add5(3)                    // 8

// 记忆化
def fib = { n ->
    n <= 1 ? n : fib(n-1) + fib(n-2)
}.memoize()

// 组合
def double = { it * 2 }
def addOne = { it + 1 }
def doubleThenAdd = double >> addOne   // 先double后addOne
def addThenDouble = double << addOne   // 先addOne后double
```

---

## 五、类与对象

### 5.1 类定义

```groovy
class Person {
    // 属性（自动生成 getter/setter）
    String name
    int age
    String email

    // 构造函数
    Person(String name, int age) {
        this.name = name
        this.age = age
    }

    // 方法
    def greet() {
        "Hello, I'm $name"
    }

    // 静态方法
    static def create(String name) {
        new Person(name, 0)
    }

    // toString
    @Override
    String toString() {
        "Person(name=$name, age=$age)"
    }
}

// 使用
def p = new Person("Tom", 25)
p.name                     // Tom
p.greet()                  // Hello, I'm Tom
println p                  // Person(name=Tom, age=25)
```

### 5.2 继承

```groovy
class Animal {
    String name
    def speak() { "..." }
}

class Dog extends Animal {
    @Override
    def speak() { "Woof!" }
    
    def fetch() { "$name fetches the ball" }
}

def dog = new Dog(name: "Rex")
dog.speak()                // Woof!
dog.fetch()                // Rex fetches the ball
```

### 5.3 接口与 Trait

```groovy
// 接口
interface Flyable {
    def fly()
}

// Trait（可以有实现）
trait Swimmable {
    def swim() { "$name is swimming" }
}

class Duck extends Animal implements Flyable, Swimmable {
    def fly() { "$name is flying" }
}

def duck = new Duck(name: "Donald")
duck.fly()                 // Donald is flying
duck.swim()                // Donald is swimming
```

### 5.4 POGO（Plain Old Groovy Object）

```groovy
// 简洁的数据类
class User {
    String name
    String email
    int age
}

// 命名参数构造
def user = new User(name: "Tom", email: "tom@example.com", age: 25)
user.name                  // Tom

// 自动生成 getter/setter
user.getName()             // Tom
user.setName("Jerry")
```

---

## 六、异常处理

```groovy
// try-catch-finally
try {
    def result = 10 / 0
} catch (ArithmeticException e) {
    println "算术错误: ${e.message}"
} catch (Exception e) {
    println "其他错误: ${e.message}"
} finally {
    println "总是执行"
}

// 多异常捕获
try {
    // ...
} catch (IOException | SQLException e) {
    println "IO或SQL错误"
}

// 抛出异常
def validate(age) {
    if (age < 0) throw new IllegalArgumentException("年龄不能为负")
}

// 自定义异常
class BusinessException extends RuntimeException {
    int code
    BusinessException(int code, String msg) {
        super(msg)
        this.code = code
    }
}
```

---

## 七、文件操作

```groovy
// 读取文件
def file = new File("data.txt")
def content = file.text                    // 读取全部
def lines = file.readLines()              // 按行读取
file.eachLine { line -> println line }    // 逐行处理

// 写入文件
new File("output.txt").text = "Hello"     // 覆盖写入
new File("output.txt") << "Hello\n"       // 追加写入
new File("output.txt").withWriter { w ->
    w.writeLine("Line 1")
    w.writeLine("Line 2")
}

// 文件操作
file.exists()                             // 是否存在
file.isFile()                             // 是否是文件
file.isDirectory()                        // 是否是目录
file.getName()                            // 文件名
file.getParent()                          // 父目录
file.length()                             // 文件大小
file.delete()                             // 删除
file.renameTo(new File("new.txt"))        // 重命名

// 目录操作
def dir = new File("mydir")
dir.mkdir()                               // 创建目录
dir.mkdirs()                              // 创建多级目录
dir.listFiles()                           // 列出文件
dir.eachFile { f -> println f.name }      // 遍历文件
dir.eachFileRecurse { f -> println f }    // 递归遍历
```

---

## 八、正则表达式

```groovy
// 定义
def regex = ~/\d+/                        // 斜杠语法
def regex2 = /\d+/                        // 简写

// 匹配
"hello123" ==~ /\w+/                      // true（完全匹配）
"hello123" =~ /\d+/                       // 部分匹配

// 查找
def matcher = "hello 123 world" =~ /\d+/
matcher.find()                            // true
matcher.group()                           // 123

// 替换
"hello world".replaceAll(/\w+/) { it.toUpperCase() }  // HELLO WORLD

// 分组
def m = "2024-03-22" =~ /(\d{4})-(\d{2})-(\d{2})/
if (m.matches()) {
    println m.group(1)                    // 2024
    println m.group(2)                    // 03
    println m.group(3)                    // 22
}
```

---

## 九、JSON 处理

```groovy
import groovy.json.*

// 解析 JSON
def json = '{"name":"Tom","age":25}'
def obj = new JsonSlurper().parseText(json)
obj.name                                  // Tom
obj.age                                   // 25

// 生成 JSON
def data = [name: "Tom", age: 25, hobbies: ["coding", "reading"]]
def jsonStr = JsonOutput.toJson(data)
// {"name":"Tom","age":25,"hobbies":["coding","reading"]}

// 格式化输出
JsonOutput.prettyPrint(jsonStr)

// 解析文件
def file = new File("data.json")
def parsed = new JsonSlurper().parse(file)
```

---

## 十、XML 处理

```groovy
// 解析 XML
def xml = '''
<users>
    <user id="1">
        <name>Tom</name>
        <age>25</age>
    </user>
</users>
'''
def root = new XmlSlurper().parseText(xml)
root.user.name.text()                     // Tom
root.user.@id.text()                      // 1

// 生成 XML
def builder = new groovy.xml.MarkupBuilder()
builder.users {
    user(id: 1) {
        name("Tom")
        age(25)
    }
}
```

---

## 十一、HTTP 请求

```groovy
// GET 请求
def url = "https://api.example.com/users"
def response = new URL(url).text

// 带参数
def params = "name=Tom&age=25"
def response2 = new URL("$url?$params").text

// POST 请求
def conn = new URL(url).openConnection()
conn.requestMethod = "POST"
conn.doOutput = true
conn.setRequestProperty("Content-Type", "application/json")
conn.outputStream.withWriter { w ->
    w.write('{"name":"Tom"}')
}
def result = conn.inputStream.text
```

---

## 十二、Groovy 特有特性

### 12.1 安全导航运算符

```groovy
// 避免 NullPointerException
def user = null
user?.name                 // null（不报错）
user?.address?.city        // null（链式安全）
```

### 12.2 Elvis 运算符

```groovy
// 简化 null 判断
def name = user?.name ?: "匿名"
// 等价于：user?.name != null ? user.name : "匿名"
```

### 12.3 展开运算符

```groovy
// 对集合中每个元素调用方法
def names = ["Tom", "Alice", "Bob"]
names*.toUpperCase()       // ["TOM", "ALICE", "BOB"]
names*.length()            // [3, 5, 3]
```

### 12.4 飞船运算符

```groovy
// 比较运算符（返回 -1, 0, 1）
1 <=> 2                    // -1
2 <=> 2                    // 0
3 <=> 2                    // 1

// 用于排序
list.sort { a, b -> a <=> b }
```

### 12.5 in 运算符

```groovy
3 in [1, 2, 3]             // true
"a" in ["a", "b"]          // true
"key" in map               // true（检查 key）
```

### 12.6 as 运算符

```groovy
// 类型转换
"123" as Integer           // 123
[1, 2, 3] as Set           // {1, 2, 3}
[name: "Tom"] as User      // User 对象
```

### 12.7 with 方法

```groovy
// 简化对象操作
def user = new User()
user.with {
    name = "Tom"
    age = 25
    email = "tom@example.com"
}
```

### 12.8 tap 方法

```groovy
// 链式操作并返回原对象
def list = [3, 1, 2].tap { it.sort() }
// list 已排序，且返回 list 本身
```

---

## 十三、GDK 扩展方法

```groovy
// 数字扩展
5.times { println it }
3.upto(7) { println it }
10.downto(1) { println it }

// 字符串扩展
"hello".capitalize()       // Hello
"hello world".tokenize()   // ["hello", "world"]
"abc" * 3                  // abcabcabc
"hello".padLeft(10)        // "     hello"
"hello".padRight(10)       // "hello     "
"hello".center(11)         // "   hello   "

// 集合扩展
[1,2,3].sum()              // 6
[1,2,3].max()              // 3
[1,2,3].min()              // 1
[1,2,3].average()          // 2.0
[1,2,3].combinations()     // 所有组合
[1,2,3].permutations()     // 所有排列
```

---

## 十四、Groovy 与 Java 互操作

```groovy
// 使用 Java 类
import java.util.ArrayList
def list = new ArrayList<String>()
list.add("hello")

// 调用 Java 方法
System.out.println("Hello")
Thread.sleep(1000)

// Java 数组
def arr = new int[5]
def arr2 = ["a", "b", "c"] as String[]

// 类型转换
def javaList = list as java.util.List
def groovyList = javaList as List
```

---

## 十五、多线程编程

### 15.1 Thread 创建与启动

```groovy
// 方式一：Thread 直接创建
def t = Thread.start {
    println "线程执行中: ${Thread.currentThread().name}"
}
t.join()

// 方式二：new Thread + start
def t2 = new Thread({
    println "Hello from thread"
})
t2.start()
t2.join()

// 带名字的线程
def t3 = Thread.start("my-thread") {
    println "执行任务"
    sleep(100)
}
t3.join()
```

### 15.2 线程池

```groovy
import java.util.concurrent.Executors

// 创建固定大小线程池
def pool = Executors.newFixedThreadPool(4)

// 提交任务
def futures = (1..10).collect { i ->
    pool.submit({
        println "任务 $i 执行在 ${Thread.currentThread().name}"
        i * 2
    } as Callable)
}

// 获取结果
futures.each { f ->
    println "结果: ${f.get()}"
}

// 关闭线程池
pool.shutdown()
pool.awaitTermination(60, java.util.concurrent.TimeUnit.SECONDS)
```

### 15.3 GPars 并行（推荐）

```groovy
// 需要添加依赖：compile 'org.codehaus.gpars:gpars:1.2.1'
import groovyx.gpars.GParsPool

// 并行 collect（map）
GParsPool.withPool {
    def result = [1, 2, 3, 4, 5].collectParallel { it * 2 }
    println result   // [2, 4, 6, 8, 10]
}

// 并行 each
GParsPool.withPool(4) {
    [1, 2, 3, 4, 5].eachParallel { println it }
}

// 并行 findAll
GParsPool.withPool {
    def evens = [1, 2, 3, 4, 5].findAllParallel { it % 2 == 0 }
    println evens   // [2, 4]
}

// 并行 reduce
GParsPool.withPool {
    def sum = [1, 2, 3, 4, 5].sumParallel { it }
    println sum    // 15
}
```

### 15.4 数据共享与同步

```groovy
// 1. synchronized 同步方法
class Counter {
    int count = 0
    
    synchronized void increment() {
        count++
    }
    
    synchronized int getCount() {
        count
    }
}

// 2. ReentrantLock
def lock = new java.util.concurrent.locks.ReentrantLock()
lock.lock()
try {
    // 临界区
    count++
} finally {
    lock.unlock()
}

// 3. AtomicInteger（推荐）
import java.util.concurrent.atomic.AtomicInteger
def atomic = new AtomicInteger(0)
atomic.incrementAndGet()      // 原子 +1
atomic.get()                  // 获取值

// 4. synchronized 块
def counter = 0
def lock2 = new Object()
synchronized(lock2) {
    counter += 1
}
```

### 15.5 线程安全集合

```groovy
import java.util.concurrent.CopyOnWriteArrayList
import java.util.concurrent.ConcurrentHashMap

// 线程安全 List
def safeList = new CopyOnWriteArrayList([1, 2, 3])
safeList.add(4)
safeList.each { /* 并安全遍历 */ }

// 线程安全 Map
def safeMap = new ConcurrentHashMap([a: 1, b: 2])
safeMap.put("c", 3)
safeMap.getOrDefault("d", 0)

// Groovy 同步包装
def syncList = Collections.synchronizedList([1, 2, 3])
def syncMap = Collections.synchronizedMap([a: 1, b: 2])
```

### 15.6 线程间通信

```groovy
// 1. CountDownLatch（倒计时门闩）
import java.util.concurrent.CountDownLatch
def latch = new CountDownLatch(3)

3.times { i ->
    Thread.start {
        println "线程 ${i} 执行"
        latch.countDown()   // 计数 -1
    }
}
latch.await()  // 等待所有线程完成
println "所有线程完成"

// 2. CyclicBarrier（循环栅栏）
import java.util.concurrent.CyclicBarrier
def barrier = new CyclicBarrier(3)

3.times { i ->
    Thread.start {
        println "线程 ${i} 到达屏障"
        barrier.await()    // 等待其他线程
        println "线程 ${i} 继续执行"
    }
}

// 3. Exchanger（交换数据）
import java.util.concurrent.Exchanger
def exchanger = new Exchanger()

Thread.start {
    def data = "数据A"
    println "线程1 交换前: $data"
    data = exchanger.exchange(data)
    println "线程1 交换后: $data"
}

Thread.start {
    def data = "数据B"
    println "线程2 交换前: $data"
    data = exchanger.exchange(data)
    println "线程2 交换后: $data"
}
```

### 15.7 Future 与 Callable

```groovy
import java.util.concurrent.*

// Callable + Future
def executor = Executors.newSingleThreadExecutor()

def future = executor.submit({ 
    sleep(1000)
    "任务完成" 
} as Callable)

println "等待结果..."
println "结果: ${future.get()}"  // 阻塞等待
println "完成: ${future.isDone()}"

executor.shutdown()

// 带超时
try {
    def result = future.get(2, TimeUnit.SECONDS)
} catch (TimeoutException e) {
    println "超时"
}

// CompletableFuture（Java 8+）
import java.util.concurrent.CompletableFuture

CompletableFuture.supplyAsync({
    "结果"
}).thenApply { it + "!" }
 .thenAccept { println it }
 .join()
```

### 15.8 线程局部变量

```groovy
// ThreadLocal
def threadLocal = new ThreadLocal<String>()
threadLocal.set("线程值")

Thread.start {
    threadLocal.set("子线程值")
    println "子线程: ${threadLocal.get()}"
}
threadLocal.get()  // 主线程值

// Groovy 闭包中的线程安全
// 注意：闭包捕获的变量是共享的，需要同步
def counter = 0
def lock = new Object()

5.times {
    Thread.start {
        synchronized(lock) {
            counter++
        }
    }
}
sleep(500)
println counter  // 5
```

### 15.9 生产者-消费者模式

```groovy
import java.util.concurrent.*

def queue = new LinkedBlockingQueue<String>(10)

// 生产者
Thread.start {
    20.times { i ->
        queue.put("item-$i")
        println "生产: item-$i"
    }
}

// 消费者
3.times { consumerId ->
    Thread.start {
        while (true) {
            def item = queue.poll(1, TimeUnit.SECONDS)
            if (item == null) break
            println "消费[$consumerId]: $item"
        }
    }
}
```

---

## 十六、速查表

### 16.1 运算符速查

| 运算符 | 说明 | 示例 |
|--------|------|------|
| `?.` | 安全导航 | `user?.name` |
| `?:` | Elvis | `name ?: "默认"` |
| `*.` | 展开 | `list*.size()` |
| `<=>` | 飞船 | `a <=> b` |
| `in` | 包含 | `3 in list` |
| `as` | 类型转换 | `"1" as Integer` |
| `**` | 幂运算 | `2 ** 10` |
| `<<` | 追加 | `list << item` |
| `=~` | 正则匹配 | `str =~ /\d+/` |
| `==~` | 完全匹配 | `str ==~ /\w+/` |

### 15.2 集合方法速查

| 方法 | 说明 |
|------|------|
| `each` | 遍历 |
| `collect` | 转换（map） |
| `findAll` | 过滤（filter） |
| `find` | 查找第一个 |
| `any` | 是否有满足 |
| `every` | 是否全满足 |
| `count` | 计数 |
| `sum` | 求和 |
| `inject` | 归约（reduce） |
| `groupBy` | 分组 |
| `sort` | 排序 |
| `unique` | 去重 |
| `flatten` | 展平 |

### 15.3 字符串速查

| 方法 | 说明 |
|------|------|
| `"${var}"` | 字符串插值 |
| `toUpperCase()` | 转大写 |
| `toLowerCase()` | 转小写 |
| `trim()` | 去空格 |
| `split()` | 分割 |
| `replace()` | 替换 |
| `contains()` | 包含 |
| `startsWith()` | 开头 |
| `endsWith()` | 结尾 |
| `length()` | 长度 |

---
