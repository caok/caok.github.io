---
layout: post
title: "socket.io"
date: 2013-11-25 19:20
categories: [NodeJs]
tags: [NodeJs]
---

{% highlight javascript %}
# SERVER(app.js)
var app = require('express').createServer()
  , io = require('socket.io').listen(app);
app.listen(80);
app.get('/', function (req, res) {
  res.sendfile(__dirname + '/index.html');
});
io.sockets.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});
{% endhighlight %}

{% highlight html %}
# CLIENT(index.html)
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io.connect('http://localhost');
  //var server = io.connect('http://' + window.location.hostname);
  socket.on('news', function (data) {
    console.log(data);
    socket.emit('my other event', { my: 'data' });
  });
</script>
{% endhighlight %}

emit: 触发事件
on: 监听事件

broadcast: 广播(触发事件)
{% highlight javascript %}
socket.broadcast.emit('event_name', data)
{% endhighlight %}

socket.io本身的事件https://github.com/LearnBoost/socket.io/wiki/Exposed-events

io.sockets.on('connection', function (socket) {});
此处的socket实际上是一个对象，它可以当作是与服务器建立连接的节点a，io.sockets代表的所有已经与服务器建立连接的节点.
所以建立“私聊”的时候，我们只要找到该节点对应的socket，然后client.emit()这样就只有该节点才能接收到事件.
当然还有中方式就是单独为这两个人开辟一个room，个人觉得这不是一个很好的办法.

比如处理私聊的时候，我们可以这样:
{% highlight javascript %}
//在一开始(用户上线时)设定下client的name为私聊做准备(将上线的用户名存储为 socket 对象的属性，以区分每个 socket 对象)
socket.name = "xxx";
.....

//clients 为存储所有连接对象的数组
var clients = io.sockets.clients();
//遍历找到该用户
clients.forEach(function (client) {
  if (client.name == data.to) {
    //触发该用户客户端的 say 事件
    client.emit('say', data);
  }
});
{% endhighlight %}

### Room
加入房间
{% highlight javascript %}
socket.join('room')
{% endhighlight %}

离开房间
{% highlight javascript %}
socket.leave('room')
{% endhighlight %}

对某个房间进行广播
{% highlight javascript %}
socket.broadcast.to('room')
or
io.sockets.in('room')
{% endhighlight %}

例如:
{% highlight javascript %}
io.sockets.on('connection', function (socket) {
  socket.broadcast.to('room').emit('event_name', data) //emit to 'room' except this socket
})
{% endhighlight %}

某个房间的成员清单
{% highlight javascript %}
io.sockets.clients('room')
{% endhighlight %}

房间：https://github.com/LearnBoost/socket.io/wiki/Rooms

讲到room的话，还有个namespaces也可以提一下

namespaces的用法:
{% highlight javascript %}
var chat = io
    .of('/chat/' + documentId)
    .on('connection', function (socket) {...}
{% endhighlight %}
或许有人会有疑惑，room和namespaces有什么区别。我也不是很清楚，哈哈，从stackoverflow上引用一段别人的来解释下:

#### This is what namespaces and rooms have in common (socket.io v0.9.8):

* 1.Both namespaces (io.of('/nsp')) and rooms (socket.join('room')) are created on the server (the latter on the fly by means of join)
* 2.Multiple namespaces and multiple rooms share the same (WebSocket) connection
* 3.The server will transmit messages over the wire only to those clients that connected to / joined a nsp / room

#### The differences:

* 1.namespaces are connected to by the client using io.connect(urlAndNsp) (the client will be added to that namespace only if it already exists on the server)
* 2.rooms can be joined only on the server side (although creating an API on the server side to enable clients to join is straightforward)
* 3.namespaces can be authorization protected
* 4.authorization is not available with rooms, but custom authorization could be added to the aforementioned, easy-to-create API on the server, in case one is bent on using rooms
* 5.rooms are part of a namespace (defaulting to the 'global' namespace)
* 6.namespaces are always rooted in the global scope

#### To not confuse the concept with the name (room or namespace), I'll use compartment to refer to the concept, and the other two names for the implementations of the concept. So if you

* 1.need per-compartment authorization, namespaces might be the easiest route to take
* 2.if you want hierarchically layered compartments (2 layers max), use a namespace/room combo
* 3.if your client-side app consists of different parts (that do not themselves care about compartments but) need to be separated from each other, use namespaces.


### 参考:
* http://socket.io
* https://github.com/learnboost/socket.io
* https://github.com/nswbmw/N-chat
* http://www.joezimjs.com/javascript/plugging-into-socket-io-the-basics
* http://www.joezimjs.com/javascript/plugging-into-socket-io-advanced
* http://sideeffect.kr:8005/
* http://stackoverflow.com/questions/10930286/socket-io-rooms-or-namespacing
* http://blog.xydudu.com/2012/08/13/socket-io.html
