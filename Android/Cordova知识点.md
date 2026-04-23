# Cordova 知识点


## 插件管理

### 插件管理命令

```bash
# 插件管理命令帮助
cordova plugin --help

# 添加插件
cordova plugin add cordova-plugin-inappbrowser

# 移除插件
cordova plugin rm cordova-plugin-inappbrowser

# 查看插件列表
cordova plugin ls 
```


## 构建运行命令

```bash 
# 构建打包安装 app
cordova build android
cordova build ios 

# 直接运行，必须先启动模拟器
cordova run android 


```