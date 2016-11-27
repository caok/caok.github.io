---
layout: post
title:  "Git Submodule"
date:   2015-01-31 22:10:57
categories: [Git]
tags: [Git]
---

### 子模块(submodule)
添加子模块：

    $ git submodule add [url] [path]

    如：$ git submodule add git@github.com:Techbay/rails_turo_in_action.git caok/rails_turo_in_action

初始化子模块：

    $ git submodule init  ----只在首次检出仓库时运行一次就行

更新子模块：

    $ git submodule update ----每次更新或切换分支后都需要运行一下

##### 删除子模块：（分4步走哦）

* 1) $ git rm --cached [path]
* 2) 编辑“.gitmodules”文件，将子模块的相关配置节点删除掉
* 3) 编辑“ .git/config”文件，将子模块的相关配置节点删除掉
* 4) 手动删除子模块残留的目录
