---
title: MacBook & Jekyll & Github Page & Chirpy静态博客搭建过程
author: huan
date: 2023-03-27 13:10:00 +0800
categories: [本站相关, 网站搭建]
tags: [Chirpy, tutorial, zh-CN]
math: true
---

> Tips：建站之初请一定首先清晰自己的诉求，从而选择合适的主题与方案来构建自己的博客网站。
>
> **个人需求**:
>  1. 界面简洁舒适
>  2. markdown文本及其中图片存储管理方便且安全
>  3. 网站方便管理且可拓展性强（包括功能和界面）
>  4. 博客更新操作快捷简便
>  5. 学习GitHub相关技术
>
> **技术栈**：
>  1. GitHub Page
>  2. GitHub Action
>  3. ruby
>  4. nodejs

## 1. 构建仓库并建立本地连接
### 1.1 Fork [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 项目来构建仓库，并将名字取为USERNAME.github.io
> 每个 GitHub 仓库都关联有一个 GitHub Pages 服务。 如果部署仓库叫作 `my-org/my-project`（`my-org` 是组织名或用户名），那么网站会被部署在 `https://my-org.github.io/my-project/` 处。 特别地，如果部署仓库叫作 `my-org/my-org.github.io`（也就是 _组织 GitHub Pages 仓库_），那么网站会被部署在 `https://my-org.github.io/`
{: .prompt-tip }

![fork light](/assets/img/blog/20230327/fork_light.png){: .w-75 .shadow .rounded-10 w='1212' h='668' }

<!-- ![fork dark](/assets/img/blog/20230327/fork_dark.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' } -->

### 1.2 创建完成后，在本地建立与github仓库的连接，具体步骤如下：
1. 创建与仓库同名的本地文件夹
	`mkdir USERNAME.github.io`
2. 进入文件夹，初始化git仓库
	`git init`
3. 添加本地与远程库的连接
	`git remote add origin git@github.com/USERNAME/USERNAME.github.io.git`
4. 若是第一次使用，则还需配置公私钥，否则直接跳转至步骤5

	4.1 设置全局配置信息（用户名和邮箱地址）
		`git config --global user.name "USERNAME"`
		`git config --global user.email "mail@xx.com" `

	4.2 生成公私钥
		`ssh-keygen -t rsa -C "mail@xx.com"`
		![generate key](/assets/img/blog/20230327/generate_key.png){: .center .w-75 .shadow .rounded-10 w='1212' h='668' }

	4.3 复制获得的公钥信息至github（添加位置：点击头像 => settings => SSH and GPG keys => New SSH key）
	
	![pubKey](/assets/img/blog/20230327/pubKey.png){: .center .shadow .rounded-5 w='672'}
	
	4.4 验证是否添加成功
		`ssh -T git@github.com`
		![sshTest](/assets/img/blog/20230327/sshTest.png){: .center .shadow .rounded-5 w='672' }
5. 使用指令拉取仓库main分支代码到本地, 指令： ~~`git pull origin main`~~

> 这里应使用git clone指令而不是git pull 否则在执行后续初始化脚本时会报错
{: .prompt-warning }

首先返回到上层文件夹，删除其中的USERNAME.github.io文件夹，并在当前目录中执行clone指令：
`git clone git@github.com:USERNAME/USERNAME.github.io.git`
	
### 1.3 安装jekyll
**遇到问题**：执行完`gem install jekyll` 命令后报错：

![gem error](/assets/img/blog/20230327/gem_error.png){: .center .shadow .rounded-10 w='700' }

这是因为 macOS 不允许您对 Mac 自带的 Ruby 版本进行任何更改。但是，像 `bundler` 这种通过使用独立版本的 Ruby（该版本不会干扰 Apple 提供的版本）来安装 gem 是可行的。
**解决方案**：重新下载一个 Ruby 版本。Mac安装Ruby有多种方式，这里选择官方推荐的通过下载ruby管理器的方式来安装Ruby，以便于今后对Ruby的拓展使用。
1. 使用Homebrew安装 `chruby` 和 `ruby-install` 
	`brew install chruby ruby-install xz`
2. 使用命令安装指定版本ruby（若不指定版本则默认为目前最新版本）
	`ruby-install ruby x.x.x
3. 配置~/.zshrc文件
```
	echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
	echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc 
	echo "chruby ruby-x.x.x" >> ~/.zshrc 
	# then run 'chruby' to see actual version
```
4. 查看ruby版本是否改变
	`ruby -v`
接着继续使用命令安装 `jekyll` 
`gem install jekyll`


## 2. 基于Chirpy主题搭建自己的博客网站

### 2.1 安装nodejs环境
> 为了运行tools文件夹中脚本文件，需要首先安装nodejs脚本，若之前安装过，可直接跳转至步骤2.2
{: .prompt-info }

1. 使用以下命令安装nvm来下载安装nodejs，方便日后nodejs的版本管理
	`brew install nvm`
2. 编辑~/.zshrc文件，添加一下环境变量
```
export NVM_DIR="$HOME/.nvm"
# This loads nvm
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  
# This loads nvm bash_completion
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  
```
3. 使用命令`source ~/.zshrc`刷新文件，并可通过`nvm --version`查看的到nvm版本号
4. 使用命令`nvm install x.x.x`安装指定版本nodejs以及对应的npm
5. 使用命令`node -v`和`npm -v`查看对应node与npm版本

>建议使用较高版本的node，经检验使用v18.15.0（对应npm为v9.5.0）能部署成功
{: .prompt-tip }

### 2.2 运行初始化脚本
使用命令`bash tools/init`，运行初始化脚本
> 该脚本会做如下几件事：
> 1.  将代码切换至最近的tag 
> 2.  从仓库中删除了:
>	- `.travis.yml`
>	- `_posts` 下的文件
>	- `docs` 目录
> 3. 配置 GitHub Actions：把 `.github/workflows/pages-deploy.yml.hook` 的后缀 `.hook` 去除，然后删除 `.github` 里的其他目录和文件。
> 4. 自动提交一个 Commit 以保存上述文件的更改。
{: .prompt-info }

### 2.3 本地部署运行
安装依赖环境
	`bundle`
本地启动jekyll服务器
	`jekyll server`
正常运行结果应如下图所示
![chirpy website](/assets/img/blog/20230327/chirpy_website.png){: .center .shadow .rounded-10 w='700' }
_网站初始化界面_

### 2.4 push本地仓库至远程仓库
若本地运行成功，即可使用指令`git push origin master`将修改后的代码发送至远程仓库
### 2.5 启用github action
在仓库中分别通过按钮Settings => Pages => Build and deployment，在下拉菜单中选择Github Actions选项，如下图所示
![github actions](/assets/img/blog/20230327/github_actions.png){: .center .shadow .rounded-10 w='700' }

接着在Actions菜单栏中即可看到脚本运行情况，如下图所示
![action deployment](/assets/img/blog/20230327/action_deployment.png){: .center .shadow .rounded-5 w='700' }
_Action脚本部署情况_

![deploy detail](/assets/img/blog/20230327/deploy_detail.png){: .center .shadow .rounded-10 w='700' }
_Action脚本部署详情_

之后即可通过地址`https://hzhang4433.github.io/`访问你的个人博客网站

>Github Action脚本解释：该脚本实现了将仓库文件自动编译部署至Github Page，并由push操作触发。在后续的网站维护中，用户只需要在本地修改添加文件并通过push操作即可直接在对应的Github Page网址中查看到修改后的结果。
>相关命令如下：
>```
> 	git add .
>		git commit -m "your commit message"
>		git push origin master
>	```
{: .prompt-info }


## 3. Chirpy主题使用文档
- Customize the Favicon：自定义网站图标
- Text and Typography：Chirpy中的mk语法及展示
- Writing a New Post：描述了如何写一篇新的博客

>至此，Chirpy主题的基本环境搭建已完成，后续将会继续更新一些有关该主题下的自定义配置的方法，敬请期待......
{: .prompt-tip }