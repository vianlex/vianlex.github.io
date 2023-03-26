---
title: location 和 rewrite 笔记
---

## location 指令
location 指令由 ngx_http_core_module 模块提供，官方文档说明
```
Syntax:	location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
Default:	—
Context:	server, location

```
The matching is performed against a normalized URI, after decoding the text encoded in the “%XX” form, 
resolving references to relative path components “.” and “..”, and possible compression of two or more adjacent slashes into a single slash. 

匹配是针对标准化的 URI 执行的，故在执行前，会将 URI 中以 `%XX` 形式编码的文本先进行解码、及解析对相对路径中 . 和 .. 组件的引用，和尽可能的去将路径中 `// ` 压缩替换成 `/`。  

A location can either be defined by a prefix string, or by a regular expression. Regular expressions are specified with the preceding “~*” modifier (for case-insensitive matching), or the “~” modifier (for case-sensitive matching). To find location matching a given request, nginx first checks locations defined using the prefix strings (prefix locations). Among them, the location with the longest matching prefix is selected and remembered. Then regular expressions are checked, in the order of their appearance in the configuration file. The search of regular expressions terminates on the first match, and the corresponding configuration is used. If no match with a regular expression is found then the configuration of the prefix location remembered earlier is used.

location 的 URI 匹配规则可以使用前缀字符串定义，也可以由正则表达式定义。使用正则表达式，前缀指定 `~*` 修饰符（用于不区分大小写的匹配）或 `~` 修饰符（适用于区分大小写匹配）。查找 location 匹配请求时，nginx 会优先检查匹配使用前缀字符串定义的 location，如果存在多个匹配的前缀字符串 location，则会取优先选择最长前缀字符串的 location 记录起来，然后按照正则表达式在配置文件中的出现顺序检查匹配正则表达式 location，搜索匹配到第一个正则表达式 location 时，就会终止查找匹配，并使用该正则表达作为最终的匹配结果。如果找不到匹配的正则表达式 location，则使用前面记录的的前缀字符串 location。

location blocks can be nested, with some exceptions mentioned below.

location 语句块是可以嵌套的，除了下面提到的一些情况是例外的。

For case-insensitive operating systems such as macOS and Cygwin, matching with prefix strings ignores a case (0.7.7). However, comparison is limited to one-byte locales.

对于 macOS 和 Cygwin 等不区分大小写的操作系统，匹配前缀字符串 location 会忽略大小写（ 0.7.7 版本）。然而，比较仅限于一个字节的区域设置。

Regular expressions can contain captures (0.7.40) that can later be used in other directives.

正则表达式可以包含捕获(0.7.40)，稍后可以在其他指令中使用。

If the longest matching prefix location has the “^~” modifier then regular expressions are not checked.

如果匹配最长的前缀字符串 location 前缀使用 ` ^~ ` 修饰符，则不会在继续检查匹配正则表达式 location。

Also, using the “=” modifier it is possible to define an exact match of URI and location. If an exact match is found, the search terminates. For example, if a “/” request happens frequently, defining “location = /” will speed up the processing of these requests, as search terminates right after the first comparison. Such a location cannot obviously contain nested locations.

当然，使用 ` = ` 修饰符定义的前缀字符串 location，它会精准匹配 URI。如果精准匹配 location 匹配上了，就会终止后续的匹配。例如，如果一个 ` / ` 频繁的请求，使用 ` = ` 符号，定义一个 ` location = / ` 精准匹配，将加快对这些请求的处理，由于匹配上第一精准的 location 之后就在终止继续查找匹配。像这样的 location 显然是不能嵌套另一个 location 的。

In versions from 0.7.1 to 0.8.41, if a request matched the prefix location without the “=” and “^~” modifiers, the search also terminated and regular expressions were not checked.

在 0.7.1 到 0.8.41 版本中，如果请求与前缀字符串的 location 匹配，但没有 `=` 和 `^~` 修饰符，也会终止检查匹配，并且不检查正则表达式。


Let’s illustrate the above by an example:

让我们通过一个例子来说明以上内容：

```
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}

```

The “/” request will match configuration A, the “/index.html” request will match configuration B, the “/documents/document.html” request will match configuration C, the “/images/1.gif” request will match configuration D, and the “/documents/1.jpg” request will match configuration E.

此 ` / ` 请求会匹配 configuration A，此 ` /index.html ` 请求会匹配 configuration B，此 `/documents/document.htm` 会匹配 configuration C，此 `/images/1.gif` 请求会匹配 configuration D，和此 `/documents/1.jpg` 请求会匹配 configuration E。

The “@” prefix defines a named location. Such a location is not used for a regular request processing, but instead used for request redirection. They cannot be nested, and cannot contain nested locations.

@ 符号前缀定义命名 location，这样定义的 location 不能用于常规请求处理，但是可以嵌套在其他 location 中用于请求重定向和转发请求，它们自身是不能嵌套其他 locations 的。

If a location is defined by a prefix string that ends with the slash character, and requests are processed by one of proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, or grpc_pass, then the special processing is performed. In response to a request with URI equal to this string, but without the trailing slash, a permanent redirect with the code 301 will be returned to the requested URI with the slash appended. If this is not desired, an exact match of the URI and location could be defined like this:

如果一个 location 的匹配规则使用前缀字符定义，并且结尾是 `/` 字符，并且请求由 proxy_pass、fastcgi_pass、uwsgi_pass、scgi_pass、memcached_pass 或 grpc_pass 其中的一个处理，则会执行响应特殊处理。

一个请求 URI 与响应（转发）的 URI 前缀路径相同，但是响应（转发）的 URI，不是以 `/` 结尾的，将会返回代码为 301 的永久重定向到 以 '/' 结尾的请求 URI。如果不希望这样做，可以定精确匹配 location 来匹配 URI，如下面的例子：

```
location /user/ {
    proxy_pass http://user.example.com;
}

location = /user {
    proxy_pass http://login.example.com;
}

```

常用例子说明：
```

```


## 参考链接
1. https://nginx.org/en/docs/http/ngx_http_core_module.html#location
