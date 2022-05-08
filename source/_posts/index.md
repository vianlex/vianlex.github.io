---
title:6
---
# GitHub Pages + GitHub Action + Hexo 搭建自动部署的静态网站

## GitHub Pages 搭建静态网站

1、静态网站的 github 仓库名称必须满足格式：用户名称.github.io，静态网站的访问地址即是仓库的名称，如下图，创建的仓库静态网页 github 仓库名为 vialex.github.io, 则要访问该仓库的静态网页，配置后浏览器访问 vianlex.github.io 即可。
![静态网站仓库名称设置说明图](/images/静态网站仓库名称设置说明图.png)

2、创建一个 ph-pages 分支， 并在仓库的 settings 中指定静态网站使用 ph-pages 分支，main 分支存放搭建 hexo 网站的源码，ph-pages 存放 hexo 编译后的静态网页源码，设置好后，浏览器访问 vianlex.github.io 会默认访问 pg-pages 分支下的 index.html 文件，静态网站使用分支设置如下图：
![静态网站分支设置图](/images/静态网站分支设置图.png)


## 手动使用 hexo deploy 部署静态网站
使用 hexo deploy 部署静态网站，其实就是把我们搭建 hexo 网站编译，并将编译后的静态网站源码推送到 pg-pages 分支,  使用 hexo deploy 部署必须安装 hexo-deployer-git 插件和修改 _config.yml 中 deploy 参数。

1、安装 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git) 插件
```
npm install hexo-deployer-git --save
```
2、修改  _config.yml 配置文件
以下配置的作用：是使用 hexo deploy 编译 hexo 源码后，将生成的静态网页，推送到指定的仓库和分支，推送的仓库类型可以是 git、gitlab 或者其他
```
deploy:
  type: git  
  repo: git@github.com:vianlex/vianlex.github.io.git  # 指定我们静态网站部署 github 仓库
  branch: gh-pages  # 因为上面我们上面设置 vianlex.github.io 静态网站指定的分支是 ph-pages，所以要将编译后的静态网站源码推动到该分支
```

## GitHub Actions 自动部署静态网站
自动部署静态网站也需要跟手动部署一样，安装好 hexo-deployer-git 插件和配置好 _config.yml 中 deploy 参数，自动部署其实就是利用 Github 提供的 Github Action 代替我们的手动执行 hexo deploy，
要想使用 Github Actions 流水线要做的几个部署如下。

1、生成部署密钥命令如下：

密钥可以使用自己以前生成过的访问 github 或者访问其他应用的 ssh 密钥，不过为安全最好是单独生成新的密钥

```
 ssh-keygen -t rsa  -f github-deploy-key

-t 指定密钥的类型，可以省略
-f 指定密钥文件名称

```

2、 在仓库的 settings 配置刚才生成的密钥，公钥和私钥的配置是为 GitHub Actions 流水线编译部署能将 hexo 源码编译的静态网站源码推送到仓库的 ph-pages 分支
2.1、设置访问仓库的公钥，如下图：
![设置访问仓库的公钥](/images/设置访问仓库的公钥.png)
2.2、设置访问仓库需要的私钥，如下图：
![设置仓库访问私钥](/images/设置仓库访问私钥.png)


3、在仓库根目录下创建 .github/workflows 目录，并目录创建一个 yml 类型的文件，文件名称随意取，然后文件内容如下：

```
# 指定 GitHub Actions 流水线的名称，即在仓库的 GitHub Actions 流水线列表中的显示的名字
name: auto-deploy-hexo-ci

on:
  # 设置 Github Actions 流水线的触发方式，使用 git push 的时候触发
  push:  
    branches:
      # 指定触发的分支，main 分支 git push 的时候触发流水线
      - main  

# 指定环境变量，方便引用，用到的时候使用 ${{ env.GIT_USER}} 引用即可
env:
  GIT_USER: vianlex
  GIT_EMAIL: zhouykwork@outlook.com
  DEPLOY_REPO: vianlex/vianlex.github.io
  DEPLOY_BRANCH: main

jobs:
  build:
    strategy:
      # 指定 node 和 os 版本 矩阵指定多版本
      matrix:
        os: [ubuntu-latest]
        node_version: [12.x]
    # ${{matrix.node_version}} 即使是引用上面定义的版本
    name: Build on node ${{ matrix.node_version }} and ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    # 流水线执行的步骤
    steps:
      # 第一步，指定拉取hexo 源码仓库分支，repository 和 ref 使用变量引用上面配置的仓库和分支
      - name: Checkout deploy repo
        uses: actions/checkout@v2
        with:
          repository: ${{ env.DEPLOY_REPO }}
          ref: ${{ env.DEPLOY_BRANCH }}
      # 第二步，指定使用的 node 版本
      - name: Use Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node_version }}
      # 第三步，设置运行环境的系统变量，并配置访问仓库的密钥，因为流水线编译 hexo 要将编译后的源码推送到仓库的 ph-pages 分支
      - name: Configuration environment
        env:
          # ${{secrets.HEXO_DEPLOY_PRI}} 表示引用的是仓库配置 settings->secrets 中的标题名称是 HEXO_DEPLOY_PRI 的私钥
          HEXO_DEPLOY_PRI: ${{secrets.HEXO_DEPLOY_PRI}}
        run: |
          sudo timedatectl set-timezone "Asia/Shanghai"
          # 在流水线的主机的用户目录下创建 .ssh 目录，并将我们在仓库设置的私钥复制到其中
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com > ~/.ssh/known_hosts
          # 配置访问仓库的用户名和邮箱
          git config --global user.name $GIT_USER
          git config --global user.email $GIT_EMAIL
      # 第四步，安装 hexo 依赖
      - name: Install dependencies
        run: |
          npm install hexo-cli -g
          npm install
      # 第五步，运行 hexo 部署命令
      - name: Hexo deploy
        run: |
          hexo clean
          hexo d

```

4、配置好后，每次 main 分支 git push 的时候 GitHub Actions 都会触发上面配置的流水线，流水线执行状态，如下图可查看：
![GitHub Actions 列表](/images/GitHub-Actions列表.png)



## 参考链接

1、https://lujiahao0708.github.io/p/df27ccfb.html
2、https://dslwind.github.io/2021-04-20-github-action-hexo