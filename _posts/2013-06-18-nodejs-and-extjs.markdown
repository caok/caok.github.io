---
layout: post
title: "nodejs and extjs"
date: 2013-06-18 15:08
comments: true
categories: [NodeJs, Extjs]
---

最近在看一些extjs和nodejs的东西，就试着将其结合着使用下。

### 1.新建express项目
首先我们新建一个express的项目，可以参照[Web Development With Express](http://caok1231.com/blog/2013/05/19/web-development-with-express/)。
```sh
express -e extjs-demo
cd extjs-demo
sudo npm install
```
增加数据库支持
```sh
sudo npm install mysql
```

<!-- more -->

连接数据库
```javascript models/db.js
var mysql = require('mysql');
 
var connection = mysql.createConnection({
    host : 'localhost',
    port : 3306,
    database : 'blog',
    user : 'root',
    password : 'yourpassword'
});
 
connection.connect();
module.exports = connection;
```
```javascript modules/article.js
var mysql = require('./db');
 
Article.get = function get(query, callback){
  var sql = 'SELECT * FROM `articles`' + (query ? (' WHERE ' + query) : '') 
 
  mysql.query(sql, function(err, rows, fields) {
    if(err){
      callback(err);
    }   
    callback(err, rows);
  }); 
};
```
这里只稍微提下，具体可参见[extjs-demo](https://github.com/caok/extjs-demo)。


### 2.引入[extjs](http://www.sencha.com/products/extjs)
我们可以从[extjs官网](http://www.sencha.com/products/extjs/download/)下载开发包，我这只取了其中的几个文件:
* resources/  extjs所有的css定义和主题都在里面
* ext-all.js  extjs的核心组件
* ext-lang-zh_CN.js  extjs的中文化部件(选用)

修改我们的主页，加载extjs
```html views/index.ejs
<head>
  <link rel="stylesheet" type="text/css" href="/javascripts/extjs/resources/css/ext-all.css" />
  <script type="text/javascript" src="/javascripts/extjs/ext-all.js"></script>
  <script type="text/javascript" src="/javascripts/extjs/ext-lang-zh_CN.js"></script>
  <script>
    Ext.Loader.setConfig({
      enabled: true
    });
  </script>
  <script type="text/javascript" src="/javascripts/app.js"> </script>
</head>
```
```javascript public/javascripts/app.js
Ext.application({
  name: "demo",
  appFolder: "/javascripts/app",  
  launch: function () {
    Ext.create('Ext.container.Viewport', {
      layout: 'fit',     
      items: [{          
        xtype: 'panel',
        title: 'Demo for Extjs',        
        html: 'you can see demo here'   
      }]                 
    });
  }
});
```
这样启动之后就能看到extjs的界面了，当然加载的extjs可能有些多，如果有经验的同学可以指点下，还有哪些东西也可以将其去掉(刚接触，要是犯了比较白的错误，希望能指出，也好学习下，哈哈)。

### 3.extjs的mvc
参照extjs官方的说明，文件结构如下定义
```
--app
  --controller
  --model
  --store
  --view
```
加入controller和view
```javascript public/javascripts/app.js
Ext.application({
  name: "demo",
  appFolder: "/javascripts/app",  
  controllers: ['Articles'],  //This is somewhat different with the previous

  launch: function () {
    Ext.create('Ext.container.Viewport', {
      layout: 'fit',     
      items: [{          
        xtype: 'panel',
        title: 'Demo for Extjs',        
        items: [{
          xtype: 'articleseditor'  //This is somewhat different with the previous
        }]
      }]                 
    });
  }
});
```
这里通过controllers: ['Articles']调用对应的controller
```javascript public/javascripts/app/controller/Articles.js
Ext.define("Demo.controller.Articles", {
  extend: 'Ext.app.Controller',

  views: ['Articles'],   //这里再取调用view

  init: function () {
    this.control({
      'articleseditor': {
        render: this.onEditorRender
      }
    });
  },

  onEditorRender: function () {
    console.log("articles editor was rendered");
  }  
});
```
```javascript public/javascripts/app/view/Articles.js
Ext.define('Demo.view.Articles', {
  extend: 'Ext.grid.Panel',
  id: "articles_editor",
  alias: 'widget.articleseditor',
 
  initComponent: function () {
    //hardcoded store with static data:
    this.store = { 
      fields: ['title', 'year'],
      data: [{
        title: 'The Matrix',
        year: '1999'
      }, {
        title: 'Star Wars: Return of the Jedi',
        year: '1983'
      }]  
    };  
    this.callParent(arguments);
  }
});
```
这里先放点假数据呈现下，至此你打开界面的时候就能看到相应的数据显示在页面上

数据放在页面上肯定是不合理的，一般model中定义数据的结构，store存取数据记录

在controller中新增
```javascript public/javascripts/app/controller/Articles.js
models: ['Articles'],
stores: ['Articles'],
```
view中修改为
```javascript public/javascripts/app/view/Articles.js
Ext.define('Demo.view.Articles', {
  extend: 'Ext.grid.Panel',
  id: "articles_editor",
  alias: 'widget.articleseditor',
     
  store: 'Articles',
  initComponent: function () {
    this.columns = [{
      header: '作者',
      dataIndex: 'user',
      flex: 1
    }, {
      header: '标题',
      dataIndex: 'title',
      flex: 1
    }, {
      header: '内容',
      dataIndex: 'content',
      flex: 1
    }, {
      header: '发生时间',
      dataIndex: 'happened_at',
      xtype : 'datecolumn',
      format : 'Y年m月d日',
      flex: 1
    }];
    this.callParent(arguments);
  }
});
```
model中定义数据类型
```javascript public/javascripts/app/model/Articles.js
Ext.define('Demo.model.Articles', {
  extend: 'Ext.data.Model',
  fields: [
    {
      name: 'id',
      type: 'string'
    }, {
      name: 'user',
      type: 'string'
    }, {
      name: 'title',
      type: 'string'
    }, {
      name: 'content',
      type: 'string'
    }, {
      name: 'happened_at',
      type: 'string'
    }]
});
```
store中存取数据记录
```javascript public/javascripts/app/model/Articles.js
Ext.define('Demo.store.Articles', {
  extend: 'Ext.data.Store',
  model: 'Demo.model.Articles',

  data: [{
    id: '1',
    user: 'tom',
    title: 'The Matrix',
    content: 'hello Tom',
    happened_at: '2013-6-1'
  }]
});
```
当然至此为止我们还没进行数据交互的部分，那接下来我们就试试跟后台的交互

### 4.数据交互
在服务器段做下相应的json数据的准备
```javascript routes/index.js
var Article = require('../models/article.js');
 
module.exports = function (app) {
  app.get('/articles', function(req, res) {
    Article.get(null, function(err, articles){
      if(err){
        articles = []; 
      }
      res.contentType('json');
      res.json({success: true, articles: articles});
    });
  });
}
```
通过proxy从服务器端的"/articles"中获取json数据
```javascript public/javascripts/app/model/Articles.js
Ext.define('Demo.store.Articles', {
  extend: 'Ext.data.Store',

  model: 'Demo.model.Articles',
  autoLoad: true,

  proxy: {
    type: 'ajax',
    url: '/articles',
    reader: {
      type: 'json',
      root: 'articles',
      successProperty: 'success'
    }
  }
});
```
这样就能在网页上显示数据库中的数据.

这里的代码可以参考[https://github.com/caok/extjs-demo](https://github.com/caok/extjs-demo)

#### 参考:
* http://docs.sencha.com/extjs/4.1.3/#!/guide/editable_grid
* http://docs.sencha.com/extjs/4.1.3/#!/guide/editable_grid_pt2
