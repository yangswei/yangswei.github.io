---
layout: post
title: "Blog 恢复"
date: 2025-02-24 
description: "从恢复博客开始，记录大模型学习历程，分享技术见解。"
tag: 大模型
---  

# 博客恢复

### 背景

从21年最后一次更新博客到现在，博客停止更新近4年，相比于基于CSDN的博客，还是用GitHub Pages搭建的博客更符合倒腾的劲，加上知识体系随着大模型的深入，需要体系的熟悉和沉淀相关知识，所以决定恢复博客。


### 恢复步骤

如果是从零开始搭建，可以参考我之前写的[博客搭建](https://blog.zhangyong.info/2021/05/23/%E5%8D
本次恢复博客，主要是环境的搭建
1. github拉取代码
2. 本地搭建jekyll环境
3. 本地运行jekyll
4. 本地修改
5. 提交代码
6. 申请与更新域名 

本地搭建jekyll环境，遇到了一些问题，但是现如今大模型很快就能解决这类问题
首先是ruby的问题，mac自带的ruby权限比较低，需要安装rvm来管理ruby版本

'''bash
brew install rbenv
rbenv install 3.2.2  # 安装最新稳定版 Ruby
rbenv global 3.2.2

gem install jekyll bundler
'''

然后就是持续报错jekyll命令找不到的问题

'''bash
gem environment    查看jekyll安装路径
- EXECUTABLE DIRECTORY: /Users/yourname/.gem/ruby/3.2.0/bin

echo 'export PATH="/opt/homebrew/lib/ruby/gems/3.1.0/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
jekyll -v
'''

解决完上述问题后，接着就是依赖问题，jekyll的依赖文件在根目录下的Gemfile

'''bash
source 'https://rubygems.org'
gem 'jekyll', '~> 4.4'
gem 'redcarpet'
gem 'jekyll-sitemap', '~> 1.0'
gem "listen", "~> 3.7"
gem 'jekyll-paginate'
'''

通过更新依赖，逐个解决
'''bash
gem install xxx
和
bundle update
'''

最后是md中引用了一段前端代码，导致jekyll编译出错，应该是jekyll对html的解析有问题，没有深究，去掉了这段代码

至此，博客本地搭建完成，接下来就是持续更新博客，和域名的申请了。

### 域名申请
因为原先域名已经存在，所以直接在阿里云续费了一年
yangshiwei.top

### 后续
新建代码分支gh-pages，用于存放静态页面
然后从github setting中设置pages，Cusom domain为yangshiwei.top

之后在域名上配置dns解析，CNAME指向yangshiwei.github.io