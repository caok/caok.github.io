---
layout: post
title: "my first gem #1"
date: 2014-02-22 22:01
comments: true
categories: [Ruby]
---

最近报名参加了hayeah的公开ruby训练营实战教程，跟着去学习如何山寨一个mongoid orm。这里把学习的过程整理成博客，多数内容来自[ruby-bootcamp-mongoid](https://github.com/hayeah/ruby-bootcamp-mongoid), 这里翻译成中文并适当加入一些自己学习过程中了解到的一些知识。

### 1.通过bundle初始化一个gem
```
bundle gem my_mongoid
```
执行命令后会自动生成一个标准的gem包结构, 初始生成的文件中，有几个是需要说明一下的：

* 1.my_mongoid.gemspec: 相当于gem的说明书，所有能看到的gem page上的信息都是来自gemspec中，Gemfile也会首先去加载 *.gemspec
* 2.lib下放我们的代码实现
* 3.lib/my_mongoid/version.rb: 指定该gem的版本

<!-- more -->

### 2.Gem release rake tasks
```
> rake -T
rake build    # Build mymongoid-0.0.1.gem into the pkg directory
rake install  # Build and install mymongoid-0.0.1.gem into system gems
rake release  # Create tag v0.0.1 and build and push mymongoid-0.0.1.gem to Rubygems
```

通过修改 mymongoid/lib/mymongoid/version.rb可以在rake -T中看出版本的变化

Gemspec 定义了一个Specification class用来包含一个gem的信息，可参考[specification-reference](http://guides.rubygems.org/specification-reference/)

修改一下gemspec中对gem的描述后，我们就可以将其打包
···
> rake build
my_mongoid 0.0.1 built to pkg/my_mongoid-0.0.1.gem.
···

### 3.Gem的组成
让我们看看ruby gem中包含些什么东西
首先生成的my_mongoid-0.0.1.gem实际上是一个POSIX tar archive
解压之后我们发现它由3个文件组成
```
> tar -tf pkg/my_mongoid-0.0.1.gem
metadata.gz
data.tar.gz
checksums.yaml.gz
```
```
> tar -xOf pkg/my_mongoid-0.0.1.gem metadata.gz | gzip -d
```
通过再次解压，我们发现metadata.gz实际上就是gemspec文件
data.tar.gz则是其他所有文件的一个打包
而checksums.yml.gz当然就是校验文件啦


### 4.安装和使用
安装gem到本地
```
gem install pkg/my_mongoid-0.0.1.gem --local
```
安装的地址
```
gem which my_mongoid
```
他怎么工作，或者说怎么说明他都ok呢
```
ruby -r my_mongoid -e 'puts MyMongoid::VERSION'
```

这样我们就拥有了一个简单的gem，尽管里面没有做任何的处理。

接下来我们跟着hayeah 的 ruby-bootcamp-mongoid 课程继续下去，不断的丰富该gem的功能



### 参考:
* http://guides.rubygems.org/
* http://guides.rubygems.org/make-your-own-gem/
* https://github.com/hayeah/ruby-bootcamp-mongoid/blob/master/00-warmup.md
* http://railscasts.com/episodes/245-new-gem-with-bundler
