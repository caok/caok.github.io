---
layout: post
title: "用octopress来写博客"
date: 2012-06-24 15:07
comments: true
categories: [Web]
tags: [Web]
---

安装
----
### 1.准备工作
首先你必须要有以下的几样东西：

> (1)git，以及github.com帐号，(我这里将blog放置在github上，没有就赶紧注册吧)
<!-- more -->

> (2)ruby的开发环境，我这使用的Octopress需要Ruby1.9.2，可以使用rbenv或rvm来简单的安装

我使用的是rbenv，简单介绍下安装：

> rbenv install 1.9.2-p290

### 2.安装Octopress
有了它你就可以像黑客一样写博客了哦！兴奋吧，o(∩∩)o...哈哈

      git clone git://github.com/imathis/octopress.git octopress
      cd octopress
      rbenv version        # 检查ruby的版本是否是1.9.2
      gem install bundler  # 安装ruby1.9.2下的bundler
      rbenv rehash         # If you use rbenv, rehash to be able to run the bundle command
      bundle install       # 安装依赖的组件
      rake install         # 安装默认的Octopress主题

我使用的是rbenv，如果是rvm，可以查看http://octopress.org/docs/setup/，上面有Octopress更加完整的安装介绍

### 3.配置github
在github上创建一个仓库，注意仓库名称要以下这种格式yourname.github.com，这样代码发布后自动这个url就可以访问了（此处一定要注意哦，我刚开始没注意，死活没得到想要的效果）。
例如你的 GitHub 帐号是 jack 就将 Repository 命名为 jack.github.com， 完成后会得到一组 GitHub Pages URL http://yourname.github.com/ (注意不能用 https协议，必须用 http协议)。

设定 GitHub Pages

    rake setup_github_pages

以上执行后会要求 read/write url for repository ：
git@github.com:yourname/yourname.github.com.git

    rake generate
    rake deploy

等待几分钟后，github上会收到一封信：“{yourname.github.com} Page build successful”，第一次发布后等比较久，之后每次都会直接更新。
当你发布之后，你就可以到 http://yourname.github.com 上看到你的博客了.

当然除了github，也可以使用HeroKu或Rsync，详细介绍：http://octopress.org/docs/deploying/

### 4.将 source 也加入 git

    git add .
    git commit -m 'initial source commit'
    git push origin source

### 5.更新 Octopress
日后有 Octopress 新版本发布，使用以下指令升级。

    git remote add octopress git://github.com/imathis/octopress.git
    git pull octopress master     # Get the latest Octopress
    bundle install                # Keep gems updated
    rake update_source            # update the template's source
    rake update_style             # update the template's style

### 6.发表新文章

    rake new_post["新文章名称"]

会在“source/_posts”目录下自动生成“Timestamp-qing-song-an-zhuang-octopress.markdown”，编辑后即可发布.

如果在zsh中出现zsh: no matches found: new_post[.....]的情况可以这么解决:

在.zshrc中加入alias rake="noglob rake"
或者rake "new_post[...]"

    rake preview

会在本地启动sinatra服务，用浏览器打开 http://localhost:4000 就可以看到效果了。如果都没有问题就可以发布了。

### 7.发布

    rake gen_deploy
    rake deploy                 #若发布后无效果可试试此命令

配置
----
### 一.配置文件说明(_config.yml)：
#### 1.Main Configs

    url: http://yoursite.com                  # blog 地址
    title: Your name                          # blog 名称，出现在左上角
    subtitle:                                 # title下面的副标题
    author:                                   # 每篇文章的posted by
    simple_search: http://google.com/search   # 右边搜索框使用的搜索引擎
    description:

#### 2.Jekyll & Plugins

    paginate: 10                    # 每页的文章数量，超过翻页
    recent_posts: 5                 # 右侧“最近发表”的模块里显示的文章数量
    excerpt_link: "Read on &rarr;"  # 在文章中使用<!-- more -->,列表页将不再显示全文，而是显示“Read on”的链接，指向全文
    default_asides: [asides/recent_posts.html, asides/github.html, asides/twitter.html]         # 用于配置侧边栏

### 二、追加新浪微博侧边栏
#### 1.先到微博秀里面生成自己的微博秀嵌入代码。(找出最后的uid和verifier)
#### 2.新建一个weibo.html放置到source/_includes/asides下

    {% if site.weibo_uid %}
      <section>
        <h1>新浪微博</h1>
        <ul id="weibo">
          <li>
            <iframe
              width="100%"
              height="550"
              class="share_self"
              frameborder="0"
              scrolling="no"
              src="http://widget.weibo.com/weiboshow/index.php?width=0&height=550&ptype={% if site.weibo_pic %}1{% else %}0{% endif %}&speed=0&skin={{weibo_skin}}&isTitle=0&noborder=1&isWeibo={% if site.weibo_show %}1{% else %}0{% endif %}&isFans={{weibo_fansline}}&uid={{site.weibo_uid}}&verifier={{site.weibo_verifier}}">
            </iframe>
          </li>
        </ul>
      </section>
    {% endif %}

#### 3.在_config.yml中增加相关的设定

    # 其它内容....
    default_asides: [asides/recent_posts.html, asides/weibo.html, asides/github.html]
    # 其它内容....
    # Weibo
    # Please refer to http://weibo.com/tool/weiboshow to get your uid and verifier.
    weibo_uid:          # 填入uid
    weibo_verifier:     # 填入verifier
    weibo_fansline: 0   # 粉丝显示多少行
    weibo_show: true    # 是否显示最近微博内容
    weibo_pic: true     # 是否显示微博中的图片
    weibo_skin: 10      # 使用哪种配色风格，数字为从1开始的微博秀风格序号
    # 其它内容....

#### 4.重新发布后就可以看到相应的效果

#### 5.在其他电脑里面同步时的操作

    git clone git@github.com:caok/caok.github.com.git
    cd caok.github.com
    git checkout source
    git clone git@github.com:caok/caok.github.com.git _deploy

### 备注
参考：http://programus.github.com/blog/2012/03/03/add-weibo-sidebar-into-octopress/

<a href="http://yanping.me/cn/blog/2012/01/07/theming-and-customization/">为Octopress修改主题和自定义样式</a>

修改网站背景或者配色后，需要rake generate才能生效

关闭github,twitter,disqus等有助于网站加载速度，特别是twitter，由于网墙的关系会使twitter.js加载特别长的时间
<a href="http://blog.jphpsf.com/2012/06/12/squeezing-octopress-for-faster-load-times">提速</a>

有兴趣还可以增加:
<a href="http://programus.github.com/blog/2012/03/04/share-weibo-button/">分享到微博</a>
<a href="https://github.com/tokkonopapa/octopress-tagcloud">标签云</a>

