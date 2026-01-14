

## 环境变量

```bash

# 查看全部环境变量，使用以下命令
printenv 或者 env 


# 设置临时环境变量
export hello=hello World

# 查看设置的临时变量
echo $hello

env | grep hello


# liunx 通过 : 符号拼接多个环境变量值，window 是通过 ; 符号
export hello=hello:world
export hello=$hello:nihao

echo $hello # 输出 hello:world:nihao



```


## 定义变量

shell 中有四种变量类型，分别为：自定义变量、环境变量、位置参数变量、预定义变量

### 自定义变量

定义变量时，= 号两边不能有空格，shell 变量的类型默认都是字符串，如果字符串没有空格时可以省略引号，否则必须加引号。

注意：单引号和双引号的区别，单引号的字符串中如果有变量，会把变量当原始字符输出

```bash

# 没有空格字符串，可以省略引号
vName=Hello 

#  单引号字符串中包含变量时，会当作原始字符
echo '$vName' # 输出 $vName
echo "$vName" # 输出 Hello 

# 字符串中包含空格，必须使用引号
vName02="Hello World"

# 打印变量值，注意：引用变量时，需要在变量名前面加 $ 符号
echo $vName $vName02

# 变量类型默认是变量字符串，字符拼接方式如下，注意字符串不支持 + 符号拼接字符
vName03=$vName$vName02Golang
echo $vName03 # 输出 Hello , 因为会默认把 vName02Golang 当前一个变量

# 可以使用空格将字符串 Golang 和 vName02 变量名隔开，但是必须使用引号 
vName03="$vName$vName02 Golang"

# 也是可以使用，以下两种方式拼接，它们的结果时等价的
vName04="$vName$vName02"Golang 
vName04=${vName}${vName02}Golang
echo $vName04 // 输出 HelloHello WorldGolang

# 将命令的运行结果赋值给变量，以下两种方式等价
usernames=$(ls /home)
usernames=`ls /home`

# 注意当 usernames 返回多个值时，会有空格，所以作为判断条件时，必须加上引号
[ "$usernames" != "xx" ] && echo "Hello World"
# 两种判断方式是等价的
test "$usernames" != "xx" && echo "Hello World 
"

# set 命令会返回所有定义定义的变量和环境变量
set | grep usernames # 查看所有命令，然后通过管道过滤出 usernames 变量

# unset 命令删除变量
unset usernames #删除已定义的变量

```

shell 中定义的变量默认都是字符串类型的，我们可以使用 declare 声明定义变量类型

declare 声明变量的语法格式为 ` declare [option] 变量名 `, option 的可选值如下：

- -i: 将变量声明为整数型
- -x: 将变量声明为环境变量
- -p: 显示指定变量被声明的类型

```bash
# 定义变量，默认是字符串
num1=90
num2=10

# 将 result 变量声明为整数类
declare -i result
# 注意 + 号两边不能有空格
result=$num1+$num2
# 只有将 result 变量声明整数类型，+ 号才起作用，不然会 + 当前字符串，结果变成（90+10）字符串
echo $result  # 输出 100

```

在 shell 中自定义变量默认是字符串，数字之间想要运算需要使用 expr 和 let 命令，或者使用 $((表达式)) 和 ${表达式} 运算式

```shell
n1=90
n2=10
# + 两边使用空格或者不使用都可以
echo $(($n1 + $n2)) # 输出 100
echo $[$n1 + $n2]

# 注意 expr 和 + 需要空格隔开
 result=$(expr $n1 + $n2)
echo $result #输出 100

# 注意 + 不能有空格
let result=$n1+$n2
echo $result #输出 100 

```


### 环境变量

```shell 

# 设置环境变量
export name=HelloWorld

# 查看所有环境变量
env

# unset 删除环境变量
unset name

```


### 位置参数变量

位置参数主要作用是，运行脚本时，可以通过参数的位置来，获取脚本传递的参数。

|位置参数变量 | 作用 |
| -- | -- |
| $n | n为数字，$代表命令本身，$1到$9代表第1-9个参数，10以上的参数需要用大括号包含，例如${10}  |
| $* | 获取所有参数，把所有的参数看作一个整体，即一个字符串  |
| $@ | 获取所有参数，返回的是一个参数列表  |
| $# | 获取传递参数总个数  |

测试例子，创建一个名为 test.sh 的测试脚本

```shell
echo ------------
echo $0
echo ------------
echo $1,$2,$3
echo ------------
echo $*
echo ------------
echo $@
echo ------------
echo $#
echo ------------

```

运行测试脚本并传递参数

```shell
sh test.sh 10 100 200 

```

结果输出为：

```bash
------------
test.sh
------------
90,100,200
------------
90 100 200
------------
90 100 200
------------
3
------------

```

### 预定义变量

|预定义变量 | 作用 |
| -- | -- |
| $? | 调用 $? 表示，获取前一条命令的运行结果，0 表示执行成功，非 0 表示执行失败 | 
| $$ | 获取当前命令运行的进程号 | 
| $1 | 获取后台运行的最后一个进程的进程号 | 

```bash

# 运行 ls -il 命令
ls -il 
# 输出 $? 前一条命令运行的结果
echo $? 

```


## 条件判断类型

Shell 支持多个类型的条件判断，并且判断符号的两边都是必须有空格的。Shell 中支持条件判断语法如下：

```bash

# 第一种写法，注意 [] 符号和条件判断表达式两边必须有空格
[ 条件判断表达式 ]

# bash 终端输入以下语句，即可测试
[ hello == hello ] && echo "Hello World"
[ hello != hello ] && echo "Hello World"

# 第二种写法 
[[ hello == hello ]] && echo "Hello World"


# 第三种写法，注意 test 和 条件判断表达式必须有空格
test 条件判断表达式

# bash 终端输入以下语句，即可测试
test "Hello" ==  "Hello" && echo "Hello test" # 输出 Hello test
test "Hello" ==  "HelloWorld" && echo "Hello test" # 没有输出



```

### 字符串类型的条件判断

| 条件判断 | 描述说明 |
|  --     |  --      |
| -z str  | 判断字符串是否为空
| -n str  | 判断字符串是否非空
| str1 == str2 或者 str1 = str2 | 判断字符串是否相等
| str1 != str2 | 判断字符串是否不相等


### 整数类型的条件判断

| 条件判断 | 描述说明 |
|  --     |  --      |
| num1 -eq num2 | 判断是否等于  
| num1 -ne num2 | 判断是否不等于
| num1 -gt num2 | 判断是否大于
| num1 -lt num2 | 判断是否小于
| num1 -ge num2 | 判断是否大于等于
| num1 -le num2 | 判断是否小于等于


### 文件类型的条件判断

| 条件判断 | 描述说明 |
|  --     |  --      |
|  -b 文件 | 判断文件是否存在



### 文件权限条件判断

| 条件判断 | 描述说明 |
|  --     |  --      |
| -r 文件 | 判断文件是否存在，并且是否有读权限
| -w 文件 | 判断文件是否存在，并且是否有写权限
| -x 文件 | 判断文件是否存在，并且是否有执行权限


### 文件比较条件判断

| 条件判断 | 描述说明 |
|  --     |  --      |
| 文件1 -nt 文件2 | 判断文件1的更新修改时间是否大于文件2
| 文件1 -ot 文件2 | 判断文件1的更新修改时间是否小于文件2
| 文件1 -ef 文件2 | 判断两个文件 INode 是否相同，注意硬链接的 Inode 和源文件的 Inode 是相同的，该判断条件一般用于判断硬链接


### 与或非条件判断

| 条件判断 | 描述说明 |
|  --     |  --      |
|  -a     | 表示与    |
|  -o     | 表示或    |
|  &&     | 表示与    |
|  ||     | 表示或    |
|  !      | 表示非    |

-a 和 && 都是表示与，-o 和 || 都表示或，它们主要区别如下：

```bash

# -a 和 -o 只能写在 [] 符号中，会报错
[ hello == hello -a world == world ] && echo "Hello World" # 输出 Hello World
[ hello == hello -o hello == world ] && echo "Hello World" # 输出 Hello World


# && 和 || 不能写在 [] 符号中，会报错
[ hello == hello ] && [ world == world ] && echo "Hello World" # 输出 Hello World
[ hello == hello ] || [ hello == world ] && echo "Hello World" # 输出 Hello World

```

&& 和 || 的第二种用法，跟 js 的 && 和 || 用法类型

1. 只有在 && 左边的命令返回真，&& 右边的命令才会被执行，只要返回假后面的命令，就不会执行。
2. 只有在 || 左边的命令返回假，|| 右边的命令才会被执行，只要返回真后面的命令，就不会执行。

```bash
# 输出 Hello World
[ hello == hello ] && echo "Hello World"
# 没有输出
[ hello == world ] && echo "Hello World"

# 赋值成功
[ hello == hello ] && v1="Hello World"
echo $v1 # 输出 Hello World

# 赋值失败，第一命令为真，则不会运行后面的命令
[ hello == hello ] ||  v2="Hello World"
echo $v2 # 输出空
```


## if 语句

if 语句命令格式

```bash

# 第一种写法，如果 if 和 then 在同一行，必须使用分号隔开
if [ 条件判断 ];then

fi

# 第二种写法
if [ 条件判断 ]
then

fi

# else 写法
if [ 条件判断 ];then

else 

fi

# elseif 写法
if [ 条件判断 ];then

elif [ 条件判断 ];then

else 

fi

```



## 参考文档：
1. https://www.bookstack.cn/read/bash-tutorial/docs-archives-redirection.md