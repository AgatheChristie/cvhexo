---
title: hexo+github笔记
date: 2023-11-21 20:18:51
tags:
- hexo
categories:
- Hexo
---

# 概述

整理一些hexo相关的东西

<!--more-->

## hexo 相关


### 常用命令
```bash
hexo clean   清理
hexo s -g    编译生成 然后本地预览
hexo d -g    编译生成 然后发布到github
```


### 比较好的例子
[Alex_McAvoy大佬的博客](https://alex-mcavoy.github.io/)




### 主题NexT官网
[NexT官网](http://theme-next.iissnan.com/getting-started.html)
官网里面的git连接是老项目的已经没有人维护了
git clone https://github.com/iissnan/hexo-theme-next themes/next
请使用
git clone https://github.com/theme-next/hexo-theme-next.git themes/next

### hexo 问题列表
[问题列表](https://github.com/hexojs/hexo/issues/3816)


### hexo d 出现 Deployer not found: git
hexo d 出现 Deployer not found: git
这是因为没安装 hexo-deployer-git 插件，在站点目录下输入下面的插件安装就好了：
```bash
npm install hexo-deployer-git --save
```

### hexo使用theme出现extends ‘_layout.swig‘,import ‘_macro/post.swig‘ as post_template问题
原因是hexo在5.0之后把swig给删除了需要自己手动安装
```bash
npm i hexo-renderer-swig
```

