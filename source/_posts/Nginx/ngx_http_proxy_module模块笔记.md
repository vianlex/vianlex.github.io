---
title: ngx_http_proxy_module 模块学习笔记
---

## 介绍
The ngx_http_proxy_module module allows passing requests to another server.

ngx_http_proxy_module 模块允许转发请求到其他的服务器，即我们平时所说的反向代理。


## proxy_pass 指令
官网语法说明
```lua
Syntax:	proxy_pass URL;
Default: —
Context: location, if in location, limit_except
```
该指令用于设置请求匹配 location 后，转发的代理服务器的 protocol 和 address 以及可选 URI。protocol 可以指定使用 http 或者 https，adress 可以指定为域名或者 ip 地址和可选的端口号(没有端口则默认端口80)，使用例子如下：
```lua
# protocol:address:port
location / {
    proxy_pass http://127.0.0.1;
}

# protocol:address:port/uri
location / {
    proxy_pass http://127.0.0.1:90/test;
}

#  UNIX-domain socket 
location / {
    proxy_pass http://unix:/tmp/backend.socket:/uri/;
}

```

请求的转发规则如下：
- 如果 proxy_pass 指令带有 URI，那么当请求需要转发到服务器时，请求 URI 与 location 匹配的前缀部分字符串，将会被替换成 proxy_pass 指令中指定的 URI。
故请求的最终转发地址等于：`proxy_pass-host + request-uri.replace前缀(location-prefix-uri, proxy_pass-uri)`
  
```lua
# 转发的替换公式：http://127.0.0.1:8080 + request-uri.replace前缀(location-prefix-uri, proxy_pass-uri)，注意 request-uri 是包含请求参数的。
# 访问 http://127.0.0.1/name/ 时，nginx 会将请求 URI = /name/ 与 location 前缀匹配字符串 /name/ 匹配的替换成 proxy_pass 的 URI = /remote/ 则转发后的地址为 http://127.0.0.1:8080/remote/
# 访问 http://127.0.0.1/name/user?name=90 时，nginx 会将请求 URI 与 location 匹配的 /name/ 部分替换成 /remote/，则转发后的地址为 http://127.0.0.1:8080/remote/user?name=90
location /name/ {
    proxy_pass http://127.0.0.1:8080/remote/;
}

# 访问 http://127.0.0.1/test 时，nginx 会将请求 URI 与 location 匹配的 /test 部分替换成 proxy_pass 的 URI(/remote), 则替换后的转发 URI = /remote
# 访问 http://127.0.0.1/test/hello 时，nginx 会将请求 URI 与 location 匹配的 /test 部分替换成 proxy_pass 的 URI(/remote)，则替换后的转发 URI = /remote/hello
location /test {
    proxy_pass http://127.0.0.1:8080/remote;
}

# 访问 http://127.0.0.1/hello 时，前缀不匹配该 location
# 访问 http://127.0.0.1/hello/ 时，nginx 会将请求 URI(/hello/) 与 location 匹配的 /hello/ 部分替换成 proxy_pass 的 URI(/remote), 则替换后的转发 URI = /remote
# 访问 http://127.0.0.1/hello/world 时，nginx 会将请求 URI(/hello/) 与 location 匹配的 /hello/ 部分替换成 proxy_pass 的 URI(/remote)，则替换后的转发 URI = /remotehello
location /hello/ {
    proxy_pass http://127.0.0.1:8080/remote;
}

# 以下，是另一种表达方式

# 访问 http://127.0.0.1/test/adc?param=xxx , location 匹配到请求的 URI = /test/adc 后，nginx 会将请求 URI 中与 location 匹配的前缀字符 / 去掉，变成 test/adc 后，
# 再匹配拼接到 proxy_pass 的 URI 中，则转发后的地址是 http://127.0.0.1/pathtest/adc?param=xxx
location / {
    proxy_pass http://127.0.0.1/path;    
}

# 访问 http://127.0.0.1/get/test 时，location 匹配到请求的 URI = /get/test 后，会将请求 URI 中与 location 匹配的 /get 前缀字符串去掉，然后将剩余部分 /test，再拼接到 proxy_pass URI 中，则最终的转发地址为 http://127.0.0.1/get/path/test
location /get {
  proxy_pass http://127.0.0.1/get/path;   
}

```

- 如果 proxy_pass 指令没有带有 URI，那么当请求转发到服务器时，转发后的 URI，就是请求匹配 location 的 URI。
```lua
location /some/path/ {
    proxy_pass http://127.0.0.1;
}

# 等价于

location /some/path/ {
    proxy_pass http://127.0.0.1:8080$request_uri;
}
```
访问 http://127.0.0.1/some/path 则转发后的地址为 http://127.0.0.1:8080/some/path
访问 http://127.0.0.1/some/path/test 则转发后的地址为 http://127.0.0.1:8080/some/path/test
访问 http://127.0.0.1/some/path?param=xxx 则转发后的地址为 http://127.0.0.1:8080/some/path?param=xxx


注意以下几种情况是 proxy_pass 指令不能携带 URI 的
- 如果 location 的匹配规则使用的是正则表达，proxy_pass 不能携带 URI
```lua
# 区分大小写正则表达式匹配 URI
location ~ /path {
    # 不能携带 URI，否则会报错
    proxy_pass http://127.0.0.1;
}

# 不区分大小写正则表达式匹配 URI
location ~* /case-insensitive /path/ {
   # 不能携带 URI，否则会报错
   proxy_pass http://127.0.0.1;
}
```
- 当在 location 中使用 rewrite 指令时，proxy_pass 指令不能携带 URI
```lua
location /name/ {
    rewrite    /name/([^/]+) /users?name=$1 break;
    # rewrite 匹配不成功时，执行 proxy_pass
    proxy_pass http://127.0.0.1;
}
```

- 在具名的 location 中 proxy_pass 指令不能携带 URI
```lua
location @testname {
    # 注意不能指定 URI
    proxy_pass http://127.0.0.1;
}
```

- 在 if 条件中 proxy_pass 指令不能携带 URI
```lua

location /test {
    if ($host = "test.adb.com" ){
        # 不能携带 URI
        proxy_pass http://127.0.0.1;
    }
    if ($host = "view.adb.com" ) {
        # 不能携带 URI
        proxy_pass http://127.0.0.1;
    }
    # 这里可以
    proxy_pass http://127.0.0.1/user;
}
```



## 参考链接
1. [在线测试 nginx 链接](https://nginx-playground.wizardzines.com/)