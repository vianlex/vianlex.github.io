# ngx_http_rewrite_module 模块学习笔记

## 介绍

ngx_http_rewrite_module 主要用于改变请求的 URI，通过 PCRE 正则表达式，return 指令，以及有选择的配置条件去改变。

模块中的 break、if、return、rewrite 和 set 指令会按以下顺序执行处理：
- 在同一级别指令语句块中按配置文件中出现的位置，按顺序执行；
- 如果对一个请求 URI 使用 rewrite 重定向，限制最多只能重定向10次。

一个 URL 请求到达 nginx 时，nginx 先会顺序执行 server 指令语句块中的 ngx_http_rewrite_module 指令集合，如果执行到 break 指令会直接返回 404，执行到 return 指令也会直接返回，执行到 rewrite 如果匹配 rewrite 表达式,将会重写请求的 URI 然后使用重写后的 URI 再去匹配 location。如果 server 指令语句块中没有 ngx_http_rewrite_module 指令集合或者有匹配不到的情况下，请求的 URL 则会去匹配 location。


ngx_http_rewrite_module 模块指令日志记录，记录在 error_log 文件中的，且日志的级别需要改成 notice 级别。rewrite 指令的日志还需要开启 rewrite_log。

```
rewrite_log on;
error_log  logs/error.log notice;

``` 

## break 指令

break 指令的官网语法如下：

```
Syntax:	break;
Default:	—
Context:	server, location, if
```
break 用于终止 ngx_http_rewrite_module 指令集合,即同一个指令块作用域内的指令集合。使用例子说明如下：

```lua
http {

    # ngx_http_rewrite_module 模块中的指令的优先级比 ngx_http_core_module 模块中的指令高。
    server {

       
        # 访问 127.0.0.1/b3 时，依次顺序执行 ngx_http_rewrite_module 指令，执行 break 执行后，将无法在执行后面的  rewrite ^/b3 /test/b3 指令，故会返回 404。
        rewrite ^/b1 /test/b1;
        rewrite ^/(b2) /test/$1;
        # 执行到 break 指令, 将不会执行下面的 rewrite，因为 break 指令会终止 ngx_http_rewrite_module 模块中的指令执行
        break;
        rewrite ^/b3 /test/b3;

    
        location /test/b3 {
            add_header Content-Type 'text/html; charset=utf-8';
            return 200 "Hello My is test-b3";
        }

        location /test {
            # 注意 break 只能终止 ngx_http_rewrite_module 指令集合，所以执行 break 指令后, return 不会在执行，但是会执行 pass_proxy
            break;
            return 200 "this test break ";
            pass_proxy: 127.0.0.1/xxx/
        }     
        
        location /break {
            # 通过 break 语句，控制改走 return 还是走 pass_proxy
            if ( $uri == '/break/xx') {
                break;
            }
            # 上面的 if 指令满足后，执行 break 指令，会终止 return 执行，然后运行 pass_proxy 执行，如果条件不满足，不执行 break 则会执行 return 语句直接返回，就会执行不到 pass_proxy 指令
            return 200 "this test break ";
            pass_proxy: 127.0.0.1/break
        }
    }
}

```

## if 指令

if 指令的官方语法如下：

```
Syntax:	if (condition) { ... }
Default:	—
Context:	server, location
```

if 指令支持的判断条件如下：

1. 空字符串和 0 默认 false，在1.0.1版本之前，任何以 0开头的字符串也被认为是 false

```lua
set $var 0;
location /test {
     add_header Content-Type 'text/html; charset=utf-8';
    if ($var) {
       return 200 "this is true";
    }
    return 200 "this is flase";
}

```

2. 变量和字符串比较只支持 = 和 != 符号

```lua
location /test {
    add_header Content-Type 'text/html; charset=utf-8';
    if ($request_method = POST) {
        return 200 "this is POST method";
    }
    # 变量支持在字符串中使用
    return 200 "unkown $request_method method";
}

```

3. 支持变量和正则表达式匹配，`~` 符号表示区分大小写匹配，`~*` 表示不区分大小写匹配，正则匹配支持非运算符 `!~` 和 `!~*`，同时支持捕获组，捕获组可通过 $1..$9 变量获取值。
   如果正则表示式包含 `}` 和 `;` 符号，那么正则表达式必须使用引号包起来。

```lua
location /test {
     add_header Content-Type 'text/html; charset=utf-8';
    if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
       return 200 "id = $0";
    }
    return 200 "hello world";
}
```

4. 支持使用 `-f` 和 `!-f` 符号检查文件是否存在 

```lua
location /exists {
    add_header Content-Type 'text/html; charset=utf-8';
    if (-f C:/DevTools/nginx-1.22.1/html/index.html) {
        return 200 "file is exists";
    }
    return 200 "file is not exists";
}
```

5. 支持使用 `-d` 和 `!-d` 符号检查目录是否存在 

```lua
location /exists {
    add_header Content-Type 'text/html; charset=utf-8';
    if (-d C:/DevTools/nginx-1.22.1/html) {
        return 200 "dir is exists";
    }
    return 200 "dir is not exists";
}
```

6. 支持使用 `-e` 和 `!-e` 运算符检查文件、目录或符号链接是否存在

```lua
location /exists {
    add_header Content-Type 'text/html; charset=utf-8';
    if (-e C:/DevTools/nginx-1.22.1/html/index.html) {
        return 200 "dir is exists";
    }
    return 200 "dir is not exists";
}
```

7. 支持使用 `-x` 和 `-x` 运算符检查可执行文件

```lua
location /exec {
    add_header Content-Type 'text/html; charset=utf-8';
    if (-x C:/DevTools/nginx-1.22.1/nginx.exe) {
        return 200 "file is exec";
    }
    return 200 "file is not exec";
}
```

8. 不支持与或逻辑符号判断,但是可以通过变量拼接字符串的形式实现

```lua
set $flag  "";
location /test {
    if ($request_method = POST) {
        set $flag "$flag1";
    }
    if ($protocol = HTTP) {
        set $flag "$flag1";
    }
    if ($flag = "11") {
        return 200 "this is and ";
    }
    return 200 "hello world"
}

```



## rewrite 指令

rewrite 指令的官方语法如下：

```
Syntax:	rewrite regex replacement [flag];
Default:	—
Context:	server, location, if
```
If the specified regular expression matches a request URI, URI is changed as specified in the replacement string. The rewrite directives are executed sequentially in order of their appearance in the configuration file. It is possible to terminate further processing of the directives using flags. If a replacement string starts with “http://”, “https://”, or “$scheme”, the processing stops and the redirect is returned to a client.

如果指定的 regex 正则表达式匹配请求的 URI, URI 将会被替换成指定 replacement 字符串。重写指令按照它们在配置文件中出现的顺序依次执行。可以使用标志符来终止对指令的进一步处理。如果 repalcement 字符串以 "http://"、"https://" 或 "$scheme" 开头，则停止处理并将重定向返回给客户。

falg 可选值说明：
- last：表示 regex 正则表达式匹配到 URI 后立即停止处理当前 ngx_http_rewrite_module 指令集合，并用 replactment 后的 URI 匹配查找 location
- break：表示 regex 正则表达式匹配到 URI 停止处理当前 ngx_http_rewrite_module 指令集合，如同处理 break 指令一样
- redirect：返回一个状态是 302 的临时重定向
- permanent：返回一个状态是 301 的永久重定向

永久重定向和临时重定向的区别是：浏览器会缓存永久重定向会记录，比如浏览器第一次访问 A 链接，如果返回 301 永久重定向到 B 链接，浏览器就会缓存 A 链接到 B 链接重定向记录，当浏览器再访问 A 链接时，如果缓存中有重定向记录则会直接重定向到 B 链接。如果第一次访问返回的 302 临时重定到 B 链接，浏览器每次都会重新 A 链接由 nginx 判断是不是重定向。即永久重定向会读缓存减轻服务器压力，临时重定向不会缓存，因为是临时的，临时说明随时会改变的所以不能缓存起来。


rewrite 指令在 server 指令语句块的例子说明： 

```lua

http{

    # 注意，在同一指令语句块中所有的 rewrite 指令是一个集合，nginx 会按顺序匹配，并且优选高于 location，在 server 指令中使用 last 和 break 符号，其效果是一样的。
    server{

        # 没有使用 last 或者 break 标识符，如访问 127.0.0.1/test  会按 rewrite 指令的顺序去匹配 /test ，匹配到 ^/test 后，还会继续使用重写后的 URI 重写匹配，一直往下匹配到 test5，然后在使用 test5 去匹配查找 location 
        rewrite ^/test /test1;
        rewrite ^/test1 /test2;
        rewrite ^/test2 /test3;
        rewrite ^/test3 /test4;
        rewrite ^/test4 /test5;

     
        # 如访问 127.0.0.1/last2 时，nginx 会按顺序从上面的 ^/test 开头的 rewirte 一直到 ^/last2 时匹配成功，重写 URI 为 /last3 因为 last 标识符结尾，所以直接终止匹配，然后使用 /last3 去匹配 location，
        # 如果能匹配查找到 location 则返回结果，否则直接返回 404
        rewrite ^/last /last1 last;
        rewrite ^/last1 /last2 last;
        rewrite ^/last2 / last3 last;


        # 注意, 如果重写的 URI 使用的是 http 协议开头，则相当于临时重定向，会直接返回给浏览器，浏览器在重定向请求新的 URL。只有重写请求的 URI 部分，nginx 才会去重新执行后续的 rewrite 或者匹配查找 location
        rewrite ^/redirect http://127.0.0.1/test.com;
        rewrite ^/redirect/last http://127.0.0.1/test.com last;
        rewrite ^/redirect/break http://127.0.0.1/test.com break;


        # 如访问访问 127.0.0.1/hello 也是先匹配 rewrite 集合如果匹配不到，再匹配查找 location，如果都匹配不到的话，nginx 会直接返回 404 
        location /hello {

        }

        location /last3 {
            add_header Content-Type 'text/html; charset=utf-8';
            return 200 "this is /last2 rewrite /last3"
        }

        location /test5 {
            add_header Content-Type 'text/html; charset=utf-8';
            return 200 "this is /test rewrite /test5"
        }

    }

}

```

在 location 指令中的 rewrite 指令例子：

```lua
location /break {
    # location 中的 rewrite 使用 break 标识，如果匹配了，就会去访问 nginx 安装目录下 html 目录的 test/page 文件
    rewrite ^/break /test/page break;
    # 如果 rewrite 不匹配，执行 return 
    add_header Content-Type 'text/html; charset=utf-8';
    return 200 "can't match rewrite, so return";
}

location /last {
    # location 中的 rewrite 如果使用 last 标识，根 server 指令中的效果一致 
    rewrite ^/break /test last;
    # 匹配不通过就走 return, 因为 return 的优先级比 pass_proxy 高
    pass_proxy 127.0.0.1/hello;
    return 200 "Hello this is last";
}

```

获取正则表达的值： 

```lua
# 访问 http://127.0.0.1/download/public/media/test.mp3
location /download/ {
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 break;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  break;
    return  403;
}

```


## return 指令

return 的官方语法如下：

```
Syntax:	return code [text];
return code URL;
return URL;
Default:	—
Context:	server, location, if

```

常用例子如下：

```lua

# 单独返回状态码
location /return0 {
    return 404;
}

# 返回字符串
location /return1 {
    add_header Content-Type 'text/html; charset=utf-8';
    return 200 "request is success";
}

# 302 重定向请求
location /return2 {
    return https://baidu.com; 
    # 或者 
    return 302 https://baidu.com;
}

# 返回内置变量
location /return3 {
    return 200 "请求的 URI : $uri ，请求的IP地址：$remote_addr";
}

```

## set 指令

set 指令用于定义变量，其官方语法如下：

```
Syntax:	set $variable value;
Default:	—
Context:	server, location, if
```

常用例子如下：

```lua
http {

    set hostname xx.test.com;

    server {

        location / {

        }
    }

}

```

## 参考链接
1. https://nginx.org/en/docs/http/ngx_http_rewrite_module.html