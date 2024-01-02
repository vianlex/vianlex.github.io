# Nginx 常用命令

## 启动命令

1. linux 启动方式

```bash
# -c 指定 nginx 配置文件 
nginx -c /path/conf/nginx.conf

# -p 指定 nginx 所在路径前缀，启动时会根据前缀路径查找 conf 文件夹下的 nginx.conf 配置文件
nginx -p /path/
```

2. window 启动方式

```bash

# 注意需要 nginx 的安装目录去执行命令，如果配置了 nginx 的环境变量，默认读取的用户用户的下的配置，需要 conf、logs 等相关文件夹复制到用户目录才行
start nginx 

# 修改配置文件重新加载
nginx -s reload 

```

## 其他常用命令

```bash

# 查看帮助信息
nginx [-h | -?]


```

1. -?,-h : 查看帮助信息
2. -v : 显示版本信息
3. -V : 显示版本信息和编译配置参数
4. -t : 测试配置文件是否正确
5. -T : 测试配置文件是否正确，同时输出所有有效配置内容
6. -q : 在配置测试期间禁止显示非错误消息
7. -s signal : 向主进程发送信号，常用的信号有 stop 快速关闭, quit 正常关闭, reopen 重新打开日志文件, reload 重新加载配置文件，启动一个加载新配置文件的 Worker Process, 正常关闭一个加载旧配置文件的 Worker Process
8. -e filename : 指定 error 文件路径 
9.  -c filename : 指定配置文件路径
10. -g directives : 指定全局配置指令, 如：nginx -g "daemon off;"
