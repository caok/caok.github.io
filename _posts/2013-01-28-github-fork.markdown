---
layout: post
title: "github fork"
date: 2013-01-28 20:11
comments: true
categories: [Git]
---

Step 1: Fork the "Spoon-Knife" repo
<!-- more -->

Step 2: Clone your fork

    git clone https://github.com/username/Spoon-Knife.git
    # Clones your fork of the repo into the current directory in terminal

Step 3: Configure remotes

    cd Spoon-Knife
    # Changes the active directory in the prompt to the newly cloned "Spoon-Knife" directory
    git remote add upstream https://github.com/octocat/Spoon-Knife.git
    # Assigns the original repo to a remote called "upstream"
    git fetch upstream
    # Pulls in changes not present in your local repository, without modifying your files
    git branch -u upstream/master
    # 如果提示没有 -u，是因为 git 版本比较低，用下面这个代替
    # git branch -t --set-upstream upstream/master

    切换至upstream/master分支
    git checkout upstream/master
    从原项目中取得最新的代码
    git pull upstream upstream/master
    切换至自己的master分支
    git checkout master
    将upstream/master(原项目)中的最新代码合并到自己的master分支中，更新master分支的代码
    git merge upstream/master

修改代码时
    1.本地新建分支
    git checkout -b add_sth
    2.把本地的 add_sth 分支保存的 github 上
    git push origin add_sth
    3.删除本地 add_sth 分支
    git branch -D add_sth
    4.删除 github 上的 add_sth 分支
    git push origin :add_sth

#### 参考：
* https://help.github.com/articles/fork-a-repo
* http://happycasts.net/episodes/37
