# tengine 安装笔记

## 编译安装 nginx 淘宝定制的 

1. 官网下载 tengine
2. 解压文件 `tar zxvf tengine-2.3.3.tar.gz`
3. `cd tengine-2.3.3 `
4. `./configure` 
5. `make `
6. `sudo make install`  默认会将 tengine 安装到 /usr/local/nginx 下
7. 编译提示 `error: the HTTP rewrite module requires the PCRE library `安装 `yum -y install pcre-devel`
8. 编译提示` error: SSL modules require the OpenSSL library `安装 `yum -y install openssl openssl-devel`


注意使用 `./configure` 生成 makefile 时，可以使用 ` ./configure --help ` 查看其有哪些参数可设置，其中部分参数例子，使用如下：

```bash
# --prefix 参数指定安装目录，--add-module 指定要安装的第三方模块目录，使用 --without 禁用默认模块，使用 --with 启动默认的模块 
./configure --prefix=/usr/local/nginx --add-module=/home/sysadmin/ngx_log_if-master --add-module=xxxxx --without-stream_return_module
```

将 nginx 注册为系统服务在 /lib/systemd/system/ 目录创建 nginx.service 文件，如下：

```bash
[Unit]
Description=nginx service
After=network.target
[Service]
Type=forking
# 指定启动命令
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

nginx 服务启动的相关命令：

```
systemctl start nginx.service 启动nginx服务
systemctl stop nginx.service　停止服务      
systemctl restart nginx.service　 重新启动服务     
systemctl list-units --type=service   查看所有已启动的服务  
systemctl status nginx.service  查看服务当前状态        
systemctl enable nginx.service  设置开机自启动      
systemctl disable nginx.service 停止开机自启动 

```

## nginx1.18 以后动态添加模块

运行命令 `nginx -V` 查看 nginx 已经安装了哪些模块，运行命令后显示示例结果如下：

```
Tengine version: Tengine/2.3.2
nginx version: nginx/1.17.3
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx  --add-module=/home/sysadmin/ngx_log_if-master
```

重新使用 ./configure 生成新的 makefile 时，需要把旧 nginx 程序中的 ` configure arguments `中的参数也要带上，不然已安装或者禁止的模块会丢弃。

```
./configure --prefix=/usr/local/nginx --add-module=/home/sysadmin/ngx_log_if-master --add-module=新增第三方的模块
```

`./configure `之后，然后运行 `make` 编译生成新的 nginx 程序，注意`make `之后，不要` make install `因为 make install 是覆盖安装。
`make` 完之后新的 nginx 程序会生成在跟 configure 同级的 objs 目录中。运行 `./objs/nginx -V ` 如果看到编译参数已经变成新编辑的参数，说明编译成功。

将新编译生成的 nginx 覆盖原来的 nginx 即可，注意备份原来的 nginx