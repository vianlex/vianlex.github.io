# Dockerfile 使用说明


## docker 构建忽略文件 .dockerignore，指定构建上下文中可以复制哪些文件, 文件格式如下

```
# * 表示忽略全部文件
* 
# ! 表示可以排除哪些文件，不需要忽略 
!package.json
!nuxt.config.js 
!.nuxt 
!static 

```

## COPY 命令无法复制文件夹，则可以利用 .dockerignore 来实现复制文件夹

```
# 比如想将 static 文件夹和其目录下的文件都复制到 /path 下，是无法实现的，COPY 只会将 static 目录中的文件复制到 /path 下，static 文件夹本身无法复制
COPY static /path/

# 这种方式可以复制文件夹，如将当前构建上下文中的文件，全部复制到 /path 目录下，COPY 命令会根据 .dockerignore 文件判断哪些文件可以复制，哪些文件不能复制
COPY . /path/

```

