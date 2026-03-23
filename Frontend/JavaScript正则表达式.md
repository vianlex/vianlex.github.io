# JavaScript 正则表达式笔记

## RegExp 对象

1. 创建正则表达式的方式如下

    ```JavaScript
    let r1 = /vim/

    let rg1 = /vim/g

    let r2 = new RegExp("vim")

    let rg2 = new RegExp("vim", "g")

    let r3 = new RegExp(/vim/g)

    let r3 = new RegExp("vim"/gi)

    ```

2. 正则表达式的修饰符说明
    
- g 表示执行全局匹配（查找所有匹配而不是在第一个匹配后停止）
- i 表示执行不区分大小写的匹配
- m 表示执行多行匹配（主要适用 ^ 开头匹配，表示匹配每一行的开头）
- s 表示正则表达式使用 . 通配符时支持换行符

    ```JavaScript
   let text = `This is RegExp example, it
   is very simple, it
   is very easy to understand`

    // 匹配 text 字符串，匹配到第一个 is 后就会停止匹配
    let result1 = text.match(/is/)
    // 全局匹配 text 字符串，加 g 修饰符的，匹配到第一 is 后还会继续往下匹配，会匹配返回所有的 is 
    let result2 =  text.match(/is/g)
    // 匹配 text 字符串，匹配字符是否以 is 开头，匹配到则返回值，匹配不到则返回 null 
    text.match(/^is/)
    // 匹配 text 字符串，如果字符串有多行，则匹配字符串的每一行是否以 is 开头(注意不设置 g 全局匹配的，只要匹配到一个结果就返回，不会再继续匹配)
    text.match(/^is/mg)
    // 测试多行字符时，回车换行后，以 tab 或者空格缩进字符串的，需要将正则表达式改为如下
    text.match(/^[\t\s]*is/mg)

    // 匹配换行符号，使用以下两种方式效果是相同的
    let text = `hello
    word`
    /hello\nw/.exec(text)
    /hello.w/s.exec(text)


    ```

3. RegExp 对象常用方法说明

- exec() 测试正则表达式是否匹配，每次匹配到第一个结果就返回，匹配不到返回 null
- test() 测试正则表达式是否匹配，每次匹配到第一个结果就返回 true ，匹配不到则返回 false
- toString() 将正则表达式转为字符串

    ```JavaScript
    // exec 方法忽略 g 修饰符， 匹配到第一个结果就返回的，否则返回 null
    /Hello/ig.exec("this is first hello, this is second hello.")
    // 同一个 RegExp 对象多次执行 exec 方法时，会从上一次记录的结束匹配位置 lastIndex 开始，继续匹配，如下
    let re = /hello/ig
    re.exec("one hello, two Hello") // 第一次执行，返回第一个 hello 并且 re.lastIndex = 9
    re.exec("one hello, two Hello") // 第二次执行，从 lastIndex =9 的位置开始匹配，匹配到第二个 Hello 并返回，re.lastIndex 变 20
    console.log(result + " : " + re.lastIndex)  
    re.exec("one hello, two Hello") // 第三次执行，从 lastIndex = 20 的位置开始执行匹配，匹配不到 hello 会返回 null，没有匹配项 lastIndex 重置为 0 
    console.log(result + " : " + re.lastIndex)  
    re.exec("one hello, two Hello") // 第四次执行，从 lastIndex = 0 的位置开始匹配，返回第一个 hello 并且 re.lastIndex = 9
    console.log(result + " : " + re.lastIndex)  

    // 测试正则表达式是否匹配，是返回 true 反之则返回 false
    /Hello/ig.test("this is first hello, this is second Hello.")

    ```

4. RegExp 对象的常用属性说明

- RegExp.lastIndex 正则表达式匹配的结束位置
- RegExp.$1 ~ RegExp.$9 用于查看与正则表达式元组相匹配的子字符串，注意当我们执行正则表达式后，RegExp 对象就会将匹配元组字符串存到其中，存放的是最后一次正则表达式执行匹配的结果
- RegExp.leftContext 和 RegExp["$`"] 表示与正则表达式相匹配子字符串左侧的字符串
- RegExp.rightContext 和 RegExp["$'"] 表示与正则表达式相匹配子字符串右侧的字符串
- RegExp.lastMatch 和 RegExp['$&'] 表示与正则表达式相匹配的子字符串

    ```JavaScript 
    // 同一个正则表达式执行多次支持 exec 或者 test 时，是以上一次匹配的结束位置，开始匹配。正则表达式通过 lastIndex 记录每次结束匹配的位置
    let re = /ll/g  
    console.log(re.lastIndex) // 正则表达式默认最后匹配的位置时 0，表示从头开始匹配
    let result = re.exec("hello") // 输出 ['ll', index: 2, input: 'hello', groups: undefined] 匹配结果数组对象中的 index 属性表示开始匹配的位置
    console.log(re.lastIndex) //  匹配到 o 字符时，匹配失败 ，此时 lastIndex 为 4
    re.exec("olleh")  // 结果输出 null , 因为再次执行正则表达式时，会从上一次匹配结束的位置 lastIndex = 4 也就是会从 'h' 字符开始匹配
    re.exec("olleh") // 结果输出 ll，因为上一次匹配失败 lastIndex 会被置为 0，所以会从头开始匹配


    // 通过 RegExp.$[1-9] 方式，获取与正则表达式相匹配的子字符串的元组子串
    let re = /(hello)/g
    let result = re.exec("one hello, two hello.")
    console.log(RegExp.$1) 
    console.log(RegExp.$2) // 输出空，是因为正则表达式 re 只有一个元组，所以只有 RegExp.$1 有值

    // String.repalce 方法引用 RegExp.$[1-9] 属性时，可以忽略掉 RegExp
    "hello".replace(/(hello)/g, "RegExp.$1 World")
    "hello".replace(/(hello)/g, "$1 World")
    // 每次匹配到都会调用回调函数
    "one hello, two hello".replace(/hello/g, args => {
        console.log(args) // 输出 hello
        // 回调函数方式不能省略 RegExp 
        return RegExp.$1 + " World"
    })

    // RegExp.leftContext 和 RegExp.rightContext  
    /hello/g.exec("this is a hello world")
    console.log(RegExp['$&']) // 输出：hello
    console.log(RegExp.leftContext ) // 输出：`this is a `
    console.log(RegExp.rightContext ) // 输出：` world`

    ```

## 支持正则表达式的 String 对象方法
- search(regexp) 查找与正则表达式相匹配子串，匹配到第一个字串时，停止匹配，并返回第一个子串的起始位置，如果没有相匹配的子串则返回 -1 ，和 indexOf 的区别是 indexOf 不能使用正则
- match(regexp) 查找与正则表达式子字符串，并以数组方式返回相匹配的子字符串
- replace(regexp, replacement) 替换与正则表达式匹配子字符串
- split(regexp, limit) regexp 拆分字符串的正则表达式，limit 限制拆分的数量

    ```JavaScript
    let text = "this is first hello, this is second hello."

    // 通过正则表达式拆分，并指定不区分大小写
    text.split(/Hello/i)
    // 通过字符串拆分, 无法指定不区分大小写
    text.split("hello")

    // 查找替换字符，以下两种方式结果等价，都只是匹配替换第一次出现的子字符串，后直接停止匹配替换
    text.replace(/hello/, "HELLO-1")
    text.replace("hello","HELLO-2")
    // 全局匹配替换
    text.repalce(/hello/g, "HELLO-1")
    text.repalceAll("hello", "HELLO-1")

    // 匹配字符，以下两种方式，查找匹配到第一个结果，则会停止继续匹配，直接返回匹配结果，如果没有一个匹配则返回 null，
    text.match("hello")
    text.match(/hello/)
    // 全局匹配查找
    text.match(/hello/g)

    // 查找匹配子字符串第一次出现的起始位置
    text.search("hello")
    text.search(/hello/i)
    // 忽略全局匹配，总是返回第一个相匹配子字符串的起始位置，并且只能正向匹配
    text.search(/hello/g)
    
    ```

## 正则表达式匹配符

1. 元字符和方括号匹配说明

    | 匹配符   |  描述说明   |
    | --  | --    |
    | . | 匹配除换行符以外的所有字符。
    | ^ | 字符串开头。
    | $ | 字符串结尾。
    | \d,\D | 匹配数字，非数字，等价于 [0-9]、[^0-9]
    | \w,\W | 匹配字符，非字符，等价于 [0-9a-zA-Z_]、[^0-9a-zA-Z_]
    | \s    | 匹配任何空白字符、等价于 [\f\n\r\t\v]。
    | \S    | 匹配非任何空白字符，等价于 [^\f\n\r\t\v]。 
    | \r, \n, \t, \f, \v   | 匹配回车符，换行符，水平制表( tab 符)，分页符，垂直制表符
    | \b, \B| 匹配单词边界，非单词边界，其中的单词指的是 \w+ 即 [0-9a-zA-Z_]+ 
    | \cx   | 匹配由 x 指定的控制符，如 \cM 匹配一个回车符，注意 x 的值必须为 A-Z 或 a-z 之一。否则，将 c 视为一个原义的 'c' 字符。
    | [0-9] | 匹配 0-9 的一个数字。
    | [abc] | 匹配 a、b 或 c 中的一个字母。 
    | [a-z] | 匹配 a 到 z 中的一个字母。
    | [^abc]| 匹配除了 a、b 或 c 中的其他字母。
    | abcd|123456 | 匹配 abcd 或 123456。
    | abc(d|e)fg  | 局部 | 匹配，匹配 abcdfg 或者 abcefg。

    ```JavaScript

    // . 匹配除换行符以外的所有字符，如下
    "hello \t\s\n world".replace(/./g, '-') // 输出：--------\n------
    "hello \t\s\r\n world".replace(/./g, '-') // 输出：--------\r\n------

    // ^ 匹配字符串的开头
    "hello world".replace(/^hello/g, "Hello") // 判断字符串是否以 hello 开头，是则将其替换为 Hello，最后输出 Hello world

    // $ 匹配字符串的结尾
    "Hello world".replace(/world$/g, "World") // 输出 Hello World

    // 匹配回车符，以下两种方式等价
    "abc\rabc".replace(/abc\cMabc/,"HelloWorld") // 输出 HelloWorld
    "abc\rabc".replace(/abc\rabc/,"HelloWorld") // 输出 HelloWorld

    // | 表示或匹配
    "abcefg".match(/abc|efg/g) // 输出  ['abc', 'efg']
    "abcefg123456".match(/abcefg|123456/g) // 输出  ['abcefg', '123456']
    // 局部 | 或匹配时，需要用括号括起来
    "aaab,abc".match(/a(aa|bc)b/g) // 输出 ['aaab']
    "aaab,abcb".match(/a(aa|bc)b/g) // 输出 ['aaab', 'abcb']

    /**
     *  单词边界匹配的字符指 [0-9a-zA-Z_]，非字符指 [^0-9a-zA-Z_]
     * \b 匹配单词边界，表示匹配字符和非字符之间的间隙，即匹配左右字符类型不相同的间隙，即匹配 [0-9a-zA-Z_] 和 [^0-9a-zA-Z_] 字符之间的间隙
     * \B 匹配非单词边界，表示匹配字符和字符之间的间隙或者非字符和非字符之间的间隙，即匹配左右字符类型相同的间隙
     */ 
    // 根据上面的规则 "Hello" 可以看成 "\bH\Be\Bl\Bl\Bo\b"
    "Hello".replace(/\b/g, "#") // 输出  #Hello#
    /**
     * "Hello-" 可以看成 "\bH\Be\Bl\B\l\Bo\b-\B"
     *  H 属于字符 [0-9a-zA-Z_]，它的前面属于非字符 [^0-9a-zA-Z_] ,字符和非字符的间隙是 \b 所以 H 字符的前面是 \b
     *  H 和 e 都是字符 [0-9a-zA-Z_]，字符和字符之间的间隙是 \B 所以 H 和 e 的间隙是 \B 
     *  o 和 - 其中 o 属于字符 [0-9a-zA-Z_]，而 - 属于非字符 [^0-9a-zA-Z_] 故他们的间隙是 \b 因为字符和非字符间隙是 \b 
     *  - 属于非字符，并且它的后面不是字符即为非字符，非字符和非字符之间的间隙是 \B
     */
    "Hello-".replace(/\b/g, '#') // 输出  #Hello#-
    "Hello-".replace(/\B/g, '@') // 输出  H@e@l@l@o-@
    // "HelloWorldHello-" 可看成 "\bH\Be\Bl\Bl\Bo\BW\Bo\Br\Bl\Bd\BH\Be\Bl\Bl\Bo\b-\B"
    "HelloWorldHello-".replace(/llo\b/,'#') // 输出 'HelloWorldHe#-'
    "HelloWorldHello-".replace(/llo\B/,'#') // 输出 'He#WorldHello-'
    
    ```


2. 量词匹配说明（量词默认都是贪婪匹配）

    | 匹配符   |  描述说明   |
    | --  | --    |
    | ?  | 匹配 0 个或 1 个字符，匹配0个字符表示匹配空字符。
    | *  | 匹配零个或多个字符。
    | +  | 匹配1个或多个字符。
    | {n} | 匹配 n 个字符。
    | {n,} | 匹配 n 个以上的字符。
    | {m,n} | 最少 m 个，最多 n 个字符。

    ```JavaScript

    // 空字符，空字符的长度 ''.length = 0
    let ec = ''  

    // * 表示匹配 0 个或者多个字符，0个字符可以表示匹配空字符，故以下例子能匹配成功
    'a'.replace(/b*/g,'-') // 输出 -a- 
    'hello'.replace(/a*/g, '-') // 输出 -h-e-l-l-o-

    // 默认贪婪匹配
    "aabb".match(/a*/g) // 输出 ['aa', '', '', '']
    "$123aBC".match(/[0-9A-Za-z]*/g) // 输出：['', '123aBC', '']
    
    // 通过添加 ? 符号限定非贪婪匹配，比如以下例子非贪婪匹配则限定匹配0个字符（空字符）
    "aabb".match(/a*?/g) // 输出 ['', '', '', '', '']

    // + 表示至少要匹配到一个 b
    'a'.match(/b+/g) // 输出：null
    // 默认贪婪匹配
    "$123aBC".match(/[0-9A-Za-z]+/g) // 输出： ['123aBC']

    // 匹配 0 个 a 字符则表示匹配空字符
    "aabbcc".match(/a{0}/g) // 输出  ['', '', '', '', '', '', '']

    // 至少匹配 0 个 a 字符
    "aabbcc".match(/a{0,}/g) // 输出 ['aa', '', '', '', '', '']

    // 至少匹配 1 个 a 字符
    "aabbcc".match(/a{1,}/g)  // 输出 ['aa']


    ```

3. 正则表达式的捕获组

    在正则表达式中可以使用括号 "()" 对正则表达式进行分组
    
    | 匹配符 | 描述说明 |
    |  --    | -- | 
    | (abc) | 匹配 abc 并捕获（记录）结果，称为捕获组，abc 子表达式可以以组序号的方式引用如 /(hello)\1/ == /(hello)hello/ 
    | (?:abc) | 匹配 abc 但不捕获（记录）结果，称为非捕获组
    | (?<name>abc) | 通过尖括号 <> 的方式，命名捕获组的名称
    | (?=abc) | 正向查找，不记录捕获结果（非捕获组）
    | (?!=abc)| 非正向查找，也称反向查找，不记录捕获结果（非捕获组）
    | (?<=abd)| 向后查找，不记录捕获结果（非捕获组）
    | (?<!abc)| 非向后查找，不记录捕获结果（非捕获组）


    ````JavaScript

    // 将正则表达式分为三组，每一个括号一个组
    let re = /((he)(ll)o(k?))/i
    /**
     * 输出：result =  ['Hello', 'Hello', 'He', 'll', index: 4, input: 'one Hello two Hello', groups: undefined]
     * result[0] 表示正则表达式匹配到的结果
     * result[1] 表示第一分组捕获的结果
     * result[2] 表示第二个分组捕获的结果
     * result[4] 表示第三个分组捕获的结果
     * result 的 index 属性表示开始匹配的位置，可以通过 re.lastIndex 获取最后匹配的位置 
     * result 的 input 属性表示正则表达式匹配的字符串
     * result 的 groups 属性表示的是捕获组名和捕获结果的 key-value, 没有命名捕获组，则值为 undefined
     */
    let result = re.exec("one Hello two Hello")

    // 输出：['Hello', 'Hello', 'He', 'll', index: 4, input: 'one Hello two Hello', groups: undefined]
    "one Hello two Hello".match(re)

    // -g 全局匹配只返回匹配接口，不会返回捕获组
    let re = /((he)(ll)o)/g
    let result = "one hello, two hello".match(re) // 输出：[hello, hello] 

    // () 表示捕获组和 (?:) 非捕获组，不记录匹配结果
    "i am hello one".match(/(?:he)llo/) // 输出 ['hello', index: 5, input: 'i am hello one', groups: undefined] 

    // (?=abc) 表示正向匹配查找子表达式
    "i am hello two".match(/he(?=llo)/) // 输出  ['he', index: 5, input: 'i am hello two', groups: undefined]
    "i am hello two".match(/(?=he)llo/) // 输出 null 

    // (?!=abc) 表示反向匹配查找子表达式，是 (?=) 的反向
    "i am hello two".match(/(?!=he)llo/) // 输出 ['llo', index: 7, input: 'i am hello two', groups: undefined]

    // (?<=abc) 反向查找，跟 (?!=abc) 效果相同
    "i am hello two".match(/(?<=he)llo/) // 输出 ['llo', index: 7, input: 'i am hello two', groups: undefined]
    "i am hello two".match(/he(?<=llo)/) // 输出 null

    // (?<!abc) 非反向查找，即正向查找跟 (?=) 效果相同
    "i am hello two".match(/he(?<!llo)/) // 输出 ['he', index: 5, input: 'i am hello two', groups: undefined]

    ````





## 参考链接
1、https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_expressions/Character_classes
2、https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/n
3、https://regex101.com/（正则测试网站）