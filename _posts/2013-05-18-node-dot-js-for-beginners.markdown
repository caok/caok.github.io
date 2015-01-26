---
layout: post
title: "Node.js for Beginners"
date: 2013-05-18 14:58
comments: true
categories: [NodeJs]
---

刚换了份工作，需要开始接触nodejs这块的东西，正好可以好好补习下javascript相关的。nodejs还是个特别好的技术，跟传统的LAMP等架构有很大的不同。废话不多说，赶紧试下先。

### 1.install [nodejs](https://github.com/joyent/node)
{% highlight bash %}
sudo apt-get update
sudo apt-get install python-software-properties python g++ make
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
{% endhighlight %}

{% highlight bash %}
➜  ~  node -v
v0.10.6
{% endhighlight %}
恭喜你nodejs安装成功了

可以先写个hello world先，哈哈
{% highlight javascript %}
# server.js
var http = require('http');              
                                         
http.createServer(function(request, response){
  response.writeHead(200);               
  response.write("Hello, world.");                                                                                                   
  response.end();                        
}).listen(3000);                         
console.log('Listening on port 3000...');
{% endhighlight %}

{% highlight bash %}
node server.js
curl http://localhost:8080
{% endhighlight %}

看到结果没？继续

### 2.install [npm](https://github.com/isaacs/npm)
npm是一个由Node.js官方提供的第三方包管理工具，我们经常需要用到它
{% highlight bash %}
curl https://npmjs.org/install.sh | sudo sh
{% endhighlight %}

### 3.install supervisor
修改代码后每次都重启应用，这对于开发是挺不方便的，不过幸好我们有[supervisor](https://github.com/isaacs/node-supervisor)
{% highlight bash %}
sudo npm install -g supervisor
{% endhighlight %}

{% highlight bash %}
supervisor server.js
{% endhighlight %}

这样我们修改了hello.js之后就只要刷新页面就可以看到修改后的结果。

### 4.文件拆分(分离服务器端代码)
当然在正式的项目中不可能一个文件搞定所有的事，那我们就试着将上面的server.js拆分了换种写法试试。

我们先把服务器脚本放到start的函数中，然后导出这个函数
{% highlight javascript %}
# server.js
var http = require('http');
                
function start() {
  function onRequest(request, response) {
    response.writeHead(200, {'Content-Type': 'test/plain'});
    response.write("Hello world");
    response.end();
  }             
                
  http.createServer(onRequest).listen(3000);
  console.log('Listening on port 3000...');
}               
                
exports.start = start;
{% endhighlight %}

创建我们的住文件并在其中启用HTTP(这块我们写在server.js中)
{% highlight javascript %}
# index.js
var server = require('./server');
server.start(); 
{% endhighlight %}

最后我们启动服务看看效果
{% highlight bash %}
node index.js
{% endhighlight %}

### 参考：
* http://www.nodebeginner.org/index-zh-cn.html
* http://www.lupaworld.com/article-224933-1.html
