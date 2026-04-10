# Nginx 知识点

## root 和 alias 的区别

Nginx 中 root 是路径拼接，alias 是路径替换，二者决定请求 URI 如何映射到磁盘文件路径。

| 维度         | root                                      | alias                                                  |
| :----------- | :---------------------------------------- | :----------------------------------------------------- |
| 路径逻辑     | 根路径 + 完整请求 URI（追加）| 别名路径 + 去掉 location 匹配前缀后的剩余 URI（替换） |
| 适用上下文   | http、server、location                    | 仅 location                                            |
| 末尾斜杠     | 可有可无（不影响拼接）| 建议以 / 结尾（否则可能被当作文件导致错误）|
| 正则匹配     | 无需捕获组                                | 必须包含捕获组且 alias 引用（官方要求）|
| 典型场景     | URI 与目录结构一致                        | 需将 URI 映射到不同目录                                |


### 简单示例

1. root 示例

```bash 
location /img/ { 
    # 映射真实路径为：root 路径 + 完整请求 URL
    root /data/dist; 
}
```

请求 /img/top.gif → 映射到 /data/dist/img/top.gif。

2. alias 示例

```bash 
location /img/ {
    # 映射真实路径为：alias 路径 + 去掉匹配前缀的 URL
    alias /data/dist/images; 
}
```

请求 /img/top.gif → 映射到 /data/dist/images/top.gif。


3. location 正则匹配时，alias 必须引用正则捕获组，root 无此限制

```bash
# alias 正则示例
location ~ ^/img/(.+\.(gif|jpe?g|png))$ {
    # location 使用正则匹配，alias 必须使用正则捕获组
    alias /data/dist/images/$1;
}
```

请求 /img/top.gif → 映射到 /data/dist/images/top.gif。