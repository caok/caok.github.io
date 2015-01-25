---
layout: post
title: "tour of NPM"
date: 2014-04-07 08:26
comments: true
categories: [NodeJs]
---

在Nodejs中也有类似Gem的东西，让我们能在项目中轻松的对模块进行管理，它就是node包管理器(NPM)。(当然nodejs中的[nvw](https://github.com/creationix/nvm)就如同ruby中rbenv，顺带一提，哈哈)

### 1.安装和使用
#### 安装:
{% highlight bash %}
curl http://npmjs.org/install.sh | sh
{% endhighlight %}

npm 默认是从国外的源获取和下载包信息, 很多情况下都很慢，可以通过简单的 ---registry 参数, 使用国内的镜像 http://r.cnpmjs.org :
{% highlight bash %}
$ npm --registry=http://r.cnpmjs.org install koa
{% endhighlight %}

当然这个是临时使用的方法，如果要持久使用可以这么做：

修改或者新建~/.npmrc，加入以下一行 registry = http://r.cnpmjs.org


####使用
{% highlight bash %}
npm install ejs           # 安装ejs依赖包，在node_modules中
npm install express -g    # 将express包安装到全局环境中
npm install <name> --save # 安装的同时，将信息写入package.json中
npm init                  # 会引导你创建一个package.json文件，包括名称、版本、作者这些信息等
npm search realtime       # 搜索和realtime相关的模块
npm view <name>           # 查看某个包的具体信息
npm remove <name>         # 移除
npm outdated              # 检查包是否已经过时
npm prune                 # 根据package.json删除你已经安装但未depended的依赖
npm update <name>         # 更新
npm link ../mynpm         # 通过链接安装package，而不是拷贝所有内容
npm ls                    # 列出当前安装的了所有包
npm root                  # 查看当前包的安装路径
npm root -g               # 查看全局的包的安装路径
npm pack                  # 将项目打包
npm help                  # 帮助，如果要单独查看install命令的帮助，可以使用的npm help install
{% endhighlight %}

### 2.包规范
包([CommonJS](http://www.commonjs.org/)包)实际上是一个存档文件，即一个被直接打包为.tar.gz或.zip的目录文件，安装后就被解压还原为目录。

CommonJS的介绍可以看下[这里](http://raychase.iteye.com/blog/1463617), 我们熟知的NodeJS、RequireJS等实际上都是CommonJS的部分实现。NPM也是对CommonJS包规范的一种实践。对Node而言，NPM帮助完成了第三方模块的发布、安装和依赖等，以此形成了一个很好的生态系统。

#### 包结构
{% highlight bash %}
package.json:  包描述文件
bin: 存放可执行二进制文件的目录
lib: 存放javascript代码的目录
doc: 存放文档的目录
test: 存放单元测试用例的代码
{% endhighlight %}

#### 本地安装
本地安装之需为NPM指明package.json文件所在的位置即可
{% highlight bash %}
npm install <tarball file>
npm install <tarball url>
npm install <folder>
{% endhighlight %}

### 3.编写自己的包
#### 编写模块
{% highlight javascript %}
# hello.js
exports.sayHello = function () {
  return 'Hello, world.';
};
{% endhighlight %}

#### 初始化包描述文件
{% highlight bash %} 
npm init
{% endhighlight %}

通过交互式的形式填写完成包描述文件
#### 注册上传
{% highlight bash %}
npm adduser
npm publish <folder>                       # 发布
npm unpublish <package name>[@<version>]   # remove your package from the registry
{% endhighlight %}

#### 管理包权限
通常，一个包只有一个人拥有权限进行发布，可以通过npm owner管理包的拥有者
{% highlight bash %}
npm owner ls <package name>
npm owner add <user> <package name>
npm owner rm <user> <package name>
{% endhighlight %}

#### 分析包
分析当前路径下能够通过模块路径找到的所有包
{% highlight bash %} 
npm ls
{% endhighlight %}

这里有个更加详细点的nodejs的npm包例子，可以参考下，地址：[这里](http://decodize.com/javascript/build-nodejs-npm-installation-package-scratch/)

### 参考:
* http://www.devthought.com/2012/02/17/npm-tricks/
* http://tobyho.com/2012/02/09/tour-of-npm/
* http://decodize.com/javascript/build-nodejs-npm-installation-package-scratch/
* http://cnodejs.org/topic/5338c5db7cbade005b023c98
