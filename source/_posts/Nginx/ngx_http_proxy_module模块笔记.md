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
- 如果 proxy_pass 指令带有 URI，那么当请求转发到服务器时，请求匹配 location 的 URI 会被替换成 proxy_pass 指令指定的 URI
```lua
location /name/ {
    proxy_pass http://127.0.0.1:8080/remote/;
}
```
访问  http://127.0.0.1/name/ 或者访问 http://127.0.0.1/name/user 或者 http://127.0.0.1/name/user?name=xxx，转发后的地址都是 http://127.0.0.1:8080/remote/


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


注意以下两种情况是 proxy_pass 指令不能携带 URI 的
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





