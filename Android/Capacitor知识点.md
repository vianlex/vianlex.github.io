# Capacitor知识点


## 安装全局命令

```bash
npm install -g @capacitor/cli
```

## 初始化空白项目

```bash
# 1. 创建项目， 按提示输入：项目名 my-app 、包名（如 com.example.app）、框架（Vanilla/React/Vue/Angular）
npm init @capacitor/app@latest

# 2. 进入目录
cd my-app

# 3. 安装依赖
npm install

# 4. 添加 Android 平台
npm install @capacitor/android 
npx cap add android

# 5. 添加 ios 平台
npm install @capacitor/ios
npx cap add ios
```


## 集成到现有 Web 项目（Vue/React/Angular）

```bash 
# 1. 在现有目录下，执行以下命令
npm install @capacitor/core

# 2. 初始化配置（生成 capacitor.config.ts）,输入：App名、包名、Web目录（Vite：dist；Vue CLI：dist；React：build）
npx cap init 

# 3. 添加 android 平台
npm install @capacitor/android
npx cap add android

# 添加 ios 平台
npm install @capacitor/ios
npx cap add ios
```

## 构建项目并打包

```bash
# 1. 编译项目
npm run build 

# 2. 将编译后项目，同步到项目根目录下的 android 或者 ios 运行原生平台项目代码中 
npx cap sync

# 3. 启动 Android Studio 并加载当前 Capacitor 项目的 Android 原生工程
npx cap open android

# 4. 在 Android Studio 中点击以下菜单 build -> Generate App Bundles or APKs -> Generate APKs 打包应用
```

## 模拟器运行应用

```bash
# 将应用安装模拟器，并运行
npx cap run android
```