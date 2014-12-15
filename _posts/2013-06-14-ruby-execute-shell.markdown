---
layout: post
title: "ruby execute shell"
date: 2013-06-14 17:01
comments: true
categories: [Ruby]
---

这里介绍几种ruby调用shell命令的方法。
### 1.Exec方法:
Kernel#exec方法通过调用指定的命令取代当前进程：

例子：
```sh
$ irb
>> exec 'echo "hello $USER"'
   hello caok
$
```
值得注意的是，exec方法用echo命令来取代了irb进程从而退出了irb。主要的缺点是，你无法从你的ruby脚本里知道这个命令是成功还是失败。

<!-- more -->

### 2.System方法。
Kernel#system方法操作命令同上， 但是它是运行一个子shell来避免覆盖当前进程。如果命令执行成功则返回true，否则返回false。
```sh
 irb
>> system 'echo "hello $USER"'
hello caok
=> true
>>
```

### 3.反引号（Backticks，Esc键下面那个键）
```sh
$ irb
>> `date`
=> "Fri Jun 14 17:18:12 CST 2013\n"
>> $?
=> #<Process::Status: pid 4586 exit 0>
>> $?.to_i
=> 0
```
这种方法是最普遍的用法了。它也是运行在一个子shell中。

### 4.IO#popen
```sh
$ irb
>> IO.popen("date") { |f| puts f.gets }
Fri Jun 14 17:22:44 CST 2013
=> nil
```

#### 参考
* http://blackanger.blog.51cto.com/140924/43730
* http://www.open-abc.com/shell-275.html
