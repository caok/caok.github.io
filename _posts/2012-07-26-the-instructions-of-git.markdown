---
layout: post
title: "git的常用操作"
date: 2012-07-26 18:01
comments: true
categories: [Git, Ubuntu]
---

ubuntu上安装git
--------------
```sh
sudo apt-get install git-core
sudo apt-get install gitk（一个较好的git图形工具）
```
<!-- more -->


git配置
-------
用户信息

    git config --global user.name "John Doe"
    git config --global user.email johndoe@example.com

文本编辑器

    git config --global core.editor vi

差异分析工具

    git config --global merge.tool vimdiff

查看配置信息

    git config --list

获取帮助

    $ git help <verb>
    $ git <verb> --help
    $ man git-<verb>


git基础
--------
##### 自己创建一个git项目

    1.创建工作路径
    mkdir demo
    cd demo
    2.创建项目仓库
    git init
    3. 将当前项目代码commit
    git add .
    git commit -m "first commit"
    4.建立远程连接，并第一次代码上传
    $ git remote add origin git@github.com:demo.git
    $ git push origin master

##### Git的一般使用流程

    1. git status
    获取状态，看当前工程时候被修改过
    2.当存在文件的修改而没有新的文件加入时
    git commit -am "xxxx"
    3.当存在有新建的文件时，应该先将新文件加入到“仓库”中，再commit
    (1)git add .
    (2)git commit -am "xxx"
    4.git pull
    从服务器中取到最新的代码，将其与自己的代码合并，若merge出现conflict时，找到产生冲突的文件，将其修改无误后再次
    git status ----------(重新确认一遍)
    git commit -am "xxxx"
    5.git push
    将自己修改的代码传到服务器

##### 提交历史

    git log        查看提交历史
    git log –p -2  常用 -p 选项展开显示每次提交的内容差异，用 -2 则仅显示最近的两次更新
    gitk           使用图形化工具查看提交历史


远程仓库的使用
-----------------
##### 查询远程仓库

    git remote     查看当前的远程仓库
    git remote -v  显示对应的克隆地址

##### 增加远程仓库

    $ git remote
    origin
    $ git remote add pb git://github.com/paulboone/ticgit.git
    $ git remote -v
    origin  git://github.com/schacon/ticgit.git
    pb  git://github.com/paulboone/ticgit.git

##### 从远程仓库pb抓取数据到本地

    $ git fetch pb
    remote: Counting objects: 58, done.
    remote: Compressing objects: 100% (41/41), done.
    remote: Total 44 (delta 24), reused 1 (delta 0)
    Unpacking objects: 100% (44/44), done.
    From git://github.com/paulboone/ticgit
     * [new branch]      master     -> pb/master
     * [new branch]      ticgit     -> pb/ticgit

##### 对远程仓库的操作
    git push origin master     推送数据到远程仓库
    git remote show origin     查看远程仓库信息
    git remote rename pb paul  远程仓库的重命名（pb => paul）
    git remote rm paul         远程仓库paul的删除


标签
-----
##### 显示已有标签

    git tag                 显示已有的标签
    git tag -l 'v1.4.2.*'   显示以v1.4.2.x形式的标签

##### 带注释的标签

    $ git tag -a v1.4 -m 'my version 1.4'
    $ git tag
    v0.1
    v1.3
    v1.4

    $ git show v1.4
    tag v1.4
    Tagger: Scott Chacon <schacon@gee-mail.com>
    Date:   Mon Feb 9 14:45:11 2009 -0800

    my version 1.4
    commit 15027957951b64cf874c3557a0f3547bd83b3ff6
    Merge: 4a447f7... a6b4c97...
    Author: Scott Chacon <schacon@gee-mail.com>
    Date:   Sun Feb 8 19:02:46 2009 -0800

        Merge branch 'experiment'

##### 轻量级标签

    $ git tag v1.4-lw
    $ git tag
    v0.1
    v1.3
    v1.4
    v1.4-lw
    v1.5

##### 分享标签

    $ git push origin v1.5
    Counting objects: 50, done.
    Compressing objects: 100% (38/38), done.
    Writing objects: 100% (44/44), 4.56 KiB, done.
    Total 44 (delta 18), reused 8 (delta 1)
    To git@github.com:schacon/simplegit.git
    * [new tag]         v1.5 -> v1.5

##### 一次推送所有（本地新增的）标签

    $ git push origin --tags
    Counting objects: 50, done.
    Compressing objects: 100% (38/38), done.
    Writing objects: 100% (44/44), 4.56 KiB, done.
    Total 44 (delta 18), reused 8 (delta 1)
    To git@github.com:schacon/simplegit.git
     * [new tag]         v0.1 -> v0.1
     * [new tag]         v1.2 -> v1.2
     * [new tag]         v1.4 -> v1.4
     * [new tag]         v1.4-lw -> v1.4-lw
     * [new tag]         v1.5 -> v1.5


分支
-----
新建分支

    git branch issue1    新建一个分支，名字为issue1
    git checkout issue1  切换到分支issue1

等同于
> git checkout -b issue1  生存一个本地分支并切换到它

git commit -am 'fix issue1'  修改后提交

##### 分支查看

    git branch               列出当前所有分支的清单
    git branch -v            查看各个分支最后一次commit信息
    git branch --merged      查看哪些分支已被并入当前分支
    git branch --no-merged   查看尚未合并的工作

##### 基本合并

    git checkout master     切换到主分支
    git merge issue1        合并issue1到主分支
    git push origin issue1  递交到远程

##### 分支的删除

    git checkout -b issue1 origin/issue1      复制远程分支到本地
    git branch -d issue1                      删除本地分支
    git branch -D testing                     强制删除本地分支
    git push origin :issue1                   删除远程分支

##### 参考
* http://robbinfan.com/blog/34/git-common-command
* http://blog.csdn.net/wirelessqa/article/details/8572928
* http://www.worldhello.net/gotgithub/index.html
