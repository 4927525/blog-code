---
title: "hugo基本操作以及自动化部署"
date: 2021-12-13T20:53:28+08:00
Description: ""
Tags: ["hugo","blog"]
Categories: ["blog"]
DisableComments: false
---

# 序

一直以来的博客(有关技术性的)都是放在CSDN上写，从来没有过自己搭建博客的想法。
近期刚好有空，然后就想着说把过往遇到的技术点还有生活的点滴放到个人博客中。
有了这个想法，于是乎就有了下面的过程。

# 1.Hugo的安装以及基础使用

## 1.1 安装Hugo

直接去hugo的github仓库下载对应系统的二进制文件：

[https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases)

然后把二进制文件放在环境变量中，由于之前配置过**GOPATH**的**bin**到环境变量中，所有这里我直接把二进制放在**bin**文件夹下面。

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211213/image.5hbwcnkdbak0.png)

验证安装

```shell
PS D:\Hugo> hugo version
hugo v0.89.4-AB01BA6E windows/amd64 BuildDate=2021-11-17T08:24:09Z VendorInfo=gohugoio
```

## 1.2 创建项目

创建一个blog为博客项目

```shell
PS D:\Hugo\Sites> hugo new site 4927525.github.io && cd 4927525.github.io
```

## 1.3 配置主题

这里使用的是这个主题：[https://github.com/lxndrblz/anatole/](https://github.com/lxndrblz/anatole/)

```shell
# 切换到博客目录
cd 4927525.github.io
git init
git submodule add https://github.com/lxndrblz/anatole.git themes/
echo 'theme = "anatole"' >> config.toml 
```

## 1.4 编写Blog

生成一篇blog模板

```shell
hugo new post/first.md
```

## 1.5 启动server

```shell
# 启动 server 预览
hugo server
```

这个时候在浏览器输入 **http://localhost:1313** 即可查看效果

## 1.6 编译静态文件

```
hugo -D
```

执行过后就会在目录的public下生成对应的静态文件，只需要部署到自己的web服务器即可。

# # 2.hugo 自动部署

每次写好了博客都需要手动部署，简直太麻烦。这里就打算使用Github的Action进行自动化部署。整个自动化流程如下：

![](https://oss.codery.cn/images/2021/03/20210319150627.png)

## 2.1 编译

### 2.1.1 脚本

创建文件:

```shell
vim .github/workflows/main.yml
```

写入脚本内容：

```yml
name: Deploy Hugo Site to Github Pages on Master Branch

on:
  push:
    branches:
      - master

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1  # v2 does not have submodules option now
       # with:
       #   submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.81.0'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }} # 这里的 ACTIONS_DEPLOY_KEY 则是上面设置 Private Key的变量名
          external_repository: 4927525/4927525.github.io # Pages 远程仓库 
          publish_dir: "./public" #传输文件目录
          keep_files: false # remove existing files
          publish_branch: master  # 分支
          commit_message: ${{ github.event.head_commit.message }} #commit message
          exclude_assets: '' #反正排除.github文件夹
```

### 2.1.2 配置密钥

* 使用git生成**ssh key**(相当于生成对密钥)
  
  ```shell
  ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
  # You will get 2 files:
  #   gh-pages.pub (public key)
  #   gh-pages     (private key)
  ```
* 打开**blog-code**仓库的**settings**，再点击**Secrets**，然后添加刚刚生成的私钥，name为**ACTIONS_DEPLOY_KEY**
    `注意：这里要用private key，且要复制整个文件的内容`![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211213/image.1sabb2i7wb4w.png)
* 同理打开**4927525.github.io**仓库的**settings**，再点击**Deploy keys**，**Allow write access**一定要勾上，否则会无法提交
    `注意：这里要用public key，且要复制整个文件的内容`
  
  ![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211213/image.5v7i6hrtls00.png)

## 2.2 部署

### 2.1.1 脚本

创建文件:

```shell
vim static/.github/workflows/main.yml
```

写入脚本内容：

```yml
name: CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Run a multi-line script
        uses: fifsky/ssh-action@master
        with:
          command: |
            cd /www/wwwroot/4927525.github.io    #自己服务器的项目地址
            git reset --hard
            git pull    #同步
          host: ${{ secrets.REMOTE_HOST }} #服务器ip
          user: ${{ secrets.REMOTE_USER }} #服务器user
          pass: ${{ secrets.PASSWORD }} #服务器密码
```

### 2.1.2 配置密钥

打开**4927525.github.io**仓库的**settings**，再点击**Secrets**，然后一共添加三个私钥，name分别为**REMOTE_HOST**,**REMOTE_USER**,**PASSWORD**

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211213/image.7j4k0ifhvok0.png)

# 至此，本次hexo转hugo完美完事

一路上还是碰到了 好多坑，但是也还行...
