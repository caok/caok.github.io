---
layout: post
title: "web development with express"
date: 2013-05-19 15:56
comments: true
categories: [NodeJs]
---

简单介绍了下Node.js之后，势必要介绍下node.js的web开发框架[Express](https://github.com/visionmedia/express).

代码可以参见[blog_node_mysql](https://github.com/caok/blog_node_mysql),也可以参见下nswbmw的[N-blog](https://github.com/nswbmw/N-blog),可以用其当作nodejs的入门教程，讲述的挺清晰的，能从中学到不少东西。

<!-- more -->
### 1.安装Express
```sh
sudo npm install -gd express
```

### 2.创建项目
```sh
express demo
cd demo
npm install
```

### 3.启动项目
```sh
node app.js
或
supervisor app.js
```

### 4.路由控制
打开 app.js，删除 , user = require('./routes/user') （我们这里用不到 routes/user.js，同时删除这个文件）和
```
app.get('/', routes.index);
app.get('/users', user.list);
```
在 app.js 最后添加：
```
routes(app);
```
修改 index.js 如下：
```javascript
module.exports = function(app){
  app.get('/',function(req,res){
    res.render('index', { title: 'Express' });
  });
};
```

### 5.数据库(mysql)
```
npm install mysql
```
```javascript package.json
"dependencies": {
  "express": "3.2.4",
  "mysql": "2.0.0-alpha8"
}
```
```javascript models/db.js
var mysql = require('mysql');

var connection = mysql.createConnection({
    host : 'localhost',
    port : 3306,
    database : 'blog',
    user : 'xxxx',
    password : 'xxxx'
});

connection.connect();
module.exports = connection;
```
调用例子
```javascript
var mysql = require('./db');
 
function User(user){
  this.name = user.name;
  this.password = user.password;
};
 
User.prototype.save = function save(callback) {
  var post  = { 
    name:this.name,
    password: this.password
  };
  var query = mysql.query('INSERT INTO users SET ?', post, function(err, result) {
    if(err){
      callback(err);
    }   
    callback(err, result);
  });
};
 
User.get = function get(username, callback){
  mysql.query('SELECT name,password FROM users WHERE name = ?', [username], function(err, rows, fields) {
    if(err){
      callback(err);
    }
    var row = rows ? rows[0] : [];
    callback(err, row);
  });
};
 
module.exports = User;
```

### 参考：
* https://github.com/visionmedia/express
* https://github.com/visionmedia/express/wiki
* https://github.com/visionmedia/jade
* https://github.com/nswbmw/N-blog/wiki/_pages
