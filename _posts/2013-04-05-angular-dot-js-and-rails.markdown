---
layout: post
title: "Angular.js and Rails #1"
date: 2013-04-05 12:54
comments: true
categories:  [AngularJs, Rails]
---

眼下JavaScript MVC越来越流行，为了紧跟时代的步伐，也打算试着学习下。在众多的[js框架](http://blog.csdn.net/luqin1988/article/details/8701466)中如何作出选择是个挺犯难的事，我比较看中其中的两种。

AngularJS来自Google ，开发它人认为基于 DOM 的模板会得到浏览器原生支持，chrome眼下又这么火，指不定用不了多久他们的观点就会成为现实。所以还是趁早先了解下。

Ember的开发者Yehuda Katz之前开发过 jQuery 和 Rails, 它的设计参考了不少Rails和Cocoa。想来会对Rails开发者的胃口，就冲这一点也是值得一试的。

这里的话，我先了学习下AngularJS，虽然Google在Reader的事情上，令人不爽，但Google推出的东西还是挺值得关注的，谁让它是互联网的老的呢。

### 1.创建一个基本的rails应用(angularjs)
{% highlight bash %}
rails new angularjs --skip-bundle -T
cd angularjs
rails g scaffold task title:string finished:boolean
rake db:create
rake db:migrate
rm public/index.html
{% endhighlight %}

### 2.修改root
{% highlight ruby %}
# config/routes.rb
root :to => 'tasks#index'
{% endhighlight %}

### 3.引入angularjs
在rails项目中引入angularjs可以直接用gem，有两个可供选择的gem，[angularjs-rails](https://github.com/hiravgandhi/angularjs-rails)和[angular-rails](https://github.com/ludicast/angular-rails)。当然你也可以直接angular.min.js放入到app/assets/javascripts/中，这样能确保用的是最新的版本。我这里用的是稳定版1.0.6

在app/views/layout/application.html.erb中
{% highlight html %}
<html ng-app> 
{% endhighlight %}

然后在app/assets/javascripts/tasks_controller.js中写入针对angular.js的 tasks controller
{% highlight javascript %}
# app/assets/javascripts/tasks_controller.js
function TasksCtrl($scope) {
  $scope.tasks = [
    {"title":"Buy milk", "finished":false},
    {"title":"Wash car", "finished":true}
  ]
}
{% endhighlight %}

这里我们首先使用写死的数据试下效果
{% highlight html %}
# app/views/tasks/index.html.erb
<div ng-controller='TasksCtrl'>
  <ul>
    <li ng-repeat='task in tasks'>
      {\{task.title}\}, {\{task.finished}\}   #这里将'\'去除，不知道为什么，我这不写'\'就显示不出来
    </li>
  </ul>
</div>
{% endhighlight %}

这样我们就可以在页面上看到"Buy milk"和"Wash car"的字样，证明angular起到作用了。

当然正式用的不可能还是用写死的假数据，如果想要获得rails中的真实数据，最简单的方式是使用angular.js中的$http。它从tasks.json中获取数据.
{% highlight javascript %}
function TasksCtrl($scope, $http) {
  $http.get('/tasks.json').success(function(data) {
    $scope.tasks = data;
    console.log(data);
  });
}
{% endhighlight %}

下面是 angular.js 和 rails 中 resource 的对比
{% highlight bash %}
HTTP Verb      Path               action      Angular.js
GET            /photos            index       'query': {method:'GET', isArray:true}
GET            /photos/new        new
POST           /photos            create      'save': {method:'POST'}
GET            /photos/:id        show        'get': {method:'GET'}
GET            /photos/:id/edit   edit
PUT            /photos/:id        update
DELETE         /photos/:id        destroy     'remove' or 'delete': {method:'DELETE'}
{% endhighlight %}

代码地址：https://github.com/caok/angularjs-samples/tree/v0.1/angularjs

#### 参考：
* http://angularjs.org
* https://github.com/angular/angular.js
* https://github.com/hiravgandhi/angularjs-rails
* http://blog.berylliumwork.com/search/label/angular.js
* http://www.dillonbuchanan.com/programming/ruby-on-rails-angularjs-resources
