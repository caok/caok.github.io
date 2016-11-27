---
layout: post
title: "install sun jdk in ubuntu"
date: 2012-10-22 17:49
categories: [Ubuntu]
tags: [Ubuntu]
---

#### 第一步：下载jdk

    wget -c http://download.oracle.com/otn-pub/java/jdk/7/jdk-7-linux-i586.tar.gz

(注：如果下载不下来，建议使用另外的下载工具下载，然后拷贝到Linux系统上。)

#### 第二步：解压安装

    sudo tar zxvf ./jdk-7-linux-i586.tar.gz -C /usr/lib/jvm
    cd /usr/lib/jvm
    sudo mv jdk1.7.0/ java-7-sun

#### 第三步：修改环境变量

    vim ~/.bashrc

添加：

    export JAVA_HOME=/usr/lib/jvm/java-7-sun
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${JAVA_HOME}/bin:$PATH

保存退出，输入以下命令使之立即生效。

    source ~/.bashrc

#### 第四步：配置默认JDK版本
由于ubuntu中可能会有默认的JDK，如openjdk，所以，为了将我们安装的JDK设置为默认JDK版本，还要进行如下工作。

执行代码:

    sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-7-sun/bin/java 300
    sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-7-sun/bin/javac 300
    sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/java-7-sun/bin/jar 300
    sudo update-alternatives --install /usr/bin/javah javah /usr/lib/jvm/java-7-sun/bin/javah 300
    sudo update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/java-7-sun/bin/javap 300

执行代码：

    sudo update-alternatives --config java
    sudo update-alternatives --config javac

系统会列出各种JDK版本，如下所示：
有 3 个候选项可用于替换 java (提供 /usr/bin/java)。


      选择       路径                                    优先级  状态
    ------------------------------------------------------------
    * 0            /usr/lib/jvm/java-6-openjdk/jre/bin/java   1061      自动模式
      1            /usr/lib/jvm/java-6-openjdk/jre/bin/java   1061      手动模式
      2            /usr/lib/jvm/java-7-sun/bin/java           300       手动模式

要维持当前值[*]请按回车键，或者键入选择的编号：2

update-alternatives: 使用 /usr/lib/jvm/java-7-sun/bin/java 来提供 /usr/bin/java (java)，于 手动模式 中。

#### 第五步：测试

    XXX@xxxx:~$ java -version
    java version "1.7.0"
    Java(TM) SE Runtime Environment (build 1.7.0-b147)
    Java HotSpot(TM) Server VM (build 21.0-b17, mixed mode
