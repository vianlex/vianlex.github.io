---
title: Node 环境配置说明
---
## npm config 命令说明

使用 npm help config 可以查看 npm 配置帮助文档

```javascript

// 查看全局或者用户级别的配置文件路径
npm config get [ globalconfig | userconfig]
// 编辑配置全局或者用户级别的配置文件
npm config edit [userconfig | globalconfig]
// 显示当前环境的配置属性
npm config list 或者 npm config ls

```

## npm install -g [cli package]

安装全局 CLI 命令包，是需要将 node_global 添加到环境变量中, 才能直接在终端中使用

## npm 配置文件说明
使用 npm set(修改用户级别配置的命令) 和 npm golabl set(修改全局配置的命令) 命令修改配置文件，都会直接写入对应的级别的配置文件中，所以也可以直接修改配置文件也是一样的效果，npm 配置的查找顺序如下：

项目的配置文件(/path/to/user/project/.npmrc)
用户的配置文件($HOME/.npmrc)
全局配置文件($PREFIX/etc/npmrc)
npm 内置配置文件(/path/to/npm/npmrc)