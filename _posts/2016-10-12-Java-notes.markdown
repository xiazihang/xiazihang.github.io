---
layout: post
title: Java学习笔记
date:  2016-08-12 22:42:00
author: "Albert"
tags:
    - Java 
---

# Java语言概述   
1. 面向对象。  
2. 跨平台，一次编译，多个平台都可以运行。  
* * *  
# Java虚拟机（JVM）以及跨平台原理
Java之所以拥有跨平台的能力，其实背后的功劳都是因为`JVM(Java Vitual Machine)`---Java虚拟机，JVM其实也是一个软件，但这个软件并没有跨平台的能力，也就是说linux版本下的JVM在macOS下也是跑不起来。而我们编写的Java源码（.java）经过编译之后，会生成一个字节码文件（.class）。Java虚拟机的作用就是负责将这些字节码文件（.class）翻译成不同操作系统下的机器码然后运行，也就是说，在不同的平台下安装不同版本的JVM，都能将我们生成的字节码文件翻译成机器码然后在不同的平台下运行，这就是Java实现跨平台的魔法。  
* * *  
# Java的版本区分：J2SE、J2EE、J2ME的区别  
Java有三个版本，分别是`J2SE`、`J2EE`、`J2ME`，以下是详细介绍：  
1. `J2SE`：Java2 Platform Standard Edition 标准版。  
* `J2SE`是Java的标准版，包含了Java的核心类库，例如`数据库连接`、`接口定义`、`输入输出`、`网络编程`等。  
2. `J2EE`：Java2 Platform Enterprise Edition 企业版。  
* `J2EE`是功能最丰富的一个版本，它包含了`J2SE`的所有类库，主要用于开发高并发，大数据量，高访问量的网站。  
3. `J2ME`：Java2 Platform Micro Edition 微型版。  
* `J2ME`只包含了`J2SE`中的一部分类库，受平台影响较大，比较适合做嵌入式开发。  
* * * 
# Java开发环境的搭建（JDK的安装）
要进行Java开发，首先必须要安装`JDK（Java Development Kit）`---Java开发工具箱。  
`JDK`是一系列工具的集合，包括`编译Java源码`、`运行Java程序`等，例如`JVM`、`基础类库`、`编译器`、`打包工具`等。不论什么样的`Java应用服务器`，都是内置了某个版本的`JDK`，因此，掌握好`JDK`是学好Java的第一步。  
`JDK`所提供的部分工具：  
* Java编译器：javac.exe  
* Java解释器：java.exe  
* Java文档生成器：javadoc.exe  
* Java调试器：jdb.exe  
前面说到的Java版本，其实指的就是`JDK`的版本。  
`JDK`的安装就不再赘述，主要说一下Java环境变量的设置，这个环境变量在我们进行Java程序开发的时候起着非常重要的作用，如果设置不当，你的程序很有可能跑不起来。（如果你对环境变量还不了解，[请看这里](http://www.weixueyuan.net/view/6310.html)）。  
以下方法是在macOS下设置Java环境变量：  
在你的dotfile中例如（~/.zshrc）添加：`export JAVA_HOME=\`/usr/libexec/java_home -v 1.8\``。  
此时在终端输入`java -version`，如果有输出并且不报错，这表明Java的环境已经安装好了，剩下的就去选一个自己喜欢的IDE装好，开始写Java吧。  

