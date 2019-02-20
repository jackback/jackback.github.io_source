---
title: Hexo入手
date: 2017-04-07 18:01:57
tags: [blog]
categories: Others
---

#### 1. 安装Git和node.js
	从官网下载对应平台binary 包，安装（linux直接将bin lib目录的文件cp到usr/local对应目录下，Window直接点击安装）
	Ubuntu平台:
		sudo apt-get install git-core
		curl https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
		安装完成后，重启终端并执行下列命令即可安装 Node.js。
		nvm install stable
	Windows平台:
		下载并安装 https://git-scm.com/download/win
		下载并安装https://nodejs.org/en/download/
	
#### 2. 安装hexo
	npm install hexo-cli -g   //终端或者cmd命令行

<!-- more -->

#### 3. 新建blog
	hexo init blog //创建网站所需要的文件
	cd blog
	npm install	//安装依赖包
	hexo server //启动一个本都服务，预览下 http://localhost:4000

#### 4. 解决一个warning
	npm install -g npm@3.3.12
	npm install hexo-deployer-git --save

#### 5. 配置_config.yml
	language: zh-CN
	deploy:
	  type: git
	  repo: <repository url>   //如果是windows上搭建环境的话，配置http协议的可以避免ssh-key
	  branch: [branch]
	  message: [message]

#### 6. 配置git config参数
	git config --global user.name "jin"
	git config --global user.name "252648955@qq.com"
	
#### 7. 部署
	hexo deploy -generate   //hexo d -g 

	[可能遇到的错误]：
	如果不是第一次推动到https://github.com/jackback/jackback.github.io.git 仓库，需要先git clone 下来，
	然后替换下hexo d -g 生成的.deploy_git目录
	然后手动做一次git push (git bash 终端执行)，后面就可以执行hexo d -g 不是报错了

	[HTTPS方式的Git仓每次需要输入密码的问题]:
	记住密码（默认15分钟）：git config --global credential.helper cache
	自己定义时间（一小时后失效）：git config credential.helper 'cache --timeout=3600'
	永久存储密码：git config --global credential.helper store
	
#### 8. 后续常用命令
	本地预览 
		hexo g //完整命令为hexo generate，用于生成静态文件 
		hexo s //完整命令为hexo server，用于启动服务器，主要用来本地预览 
	服务器部署
		hexo g //完整命令为hexo generate，用于生成静态文件 
		hexo d //完整命令为hexo deploy，用于将本地文件发布到github上 
		或者 hexo d -g //先generate再deploy
	
	新建一篇文章
		hexo n //完整命令为hexo new，用于新建一篇文章
		hexo new post "title"     //使用source/_post 内模板新建文章


    
#### 附（参考）
https://hexo.io/docs/index.html

