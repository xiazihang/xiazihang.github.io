---
layout:   post
title:    Rebuilding Rails(译)
date:     2016-12-05 23:19:00
author:   "Albert"
tags:
    - Rails 
    - Ruby 
---

> “Get your hands dirty and build your own Ruby Web Framework” 

- - -  

0 为自己重建Rails
=====

为什么要重建Rails
-----

在软件开发的学习过程中，对任何一个技术或框架的任何一个部分而言，当你试着刨根问底，并能了解到它本身最底层的实现时，往往意味着你已经对它了如指掌了，这是货真价实的能力。只有当你足够的了解它，你才能构建它。设想一下，当你真正的去构建它的时候，是否会让你觉得这确实是一件很有价值的事情呢？

本书会指引你从头开始构建一个类Rails的框架，我们会使用和Rails中同样的Ruby特性和组织结构，最终我们会让你的框架看上去和Rails同样有趣。

Ruby on Rails以它魔术般的功能而著称，而在你使用这些Ruby特性构建了你自己的Ruby框架之后，你会恍然大悟：“原来Rails是这么做的！”

同样，Rails是一个很偏执的框架，这是Rails开发团队宣称的，不过，要是你对Rails有自己的想法怎么办呢？你可以构建一个属于你自己的类Rails框架，并且你将会有更多自由发挥的空间去给你的框架添砖加瓦。

不管你最终是想成为一个真正的Rails大师或是想构建一个属于你自己的框架，本书都将会给予你很大的帮助。

谁应该重建Rails
-----

首先，你需要了解一些Ruby知识，如果你曾经开发过一些小型的Ruby应用或者中型的Rails应用，这本书将会非常适合你。如果没有这方面的经验，没关系，准备那本著名的镐头书[《Programm
ing Ruby》](http:// ruby-doc.com/docs/ProgrammingRuby/)并在你遇到不懂的Ruby问题是查阅，会非常有帮助。不过，你还是应该具备一些最基本的Ruby知识。

如果你想要复习一下Rails的话，Michael Hartl的[《Rails Tutorials》](http://ruby.railstutorial.org/ruby-on-rails-tutorial-book)是个不错的选择，链接是它的免费HTML版，当然你也可以购买它的PDF或者演示文稿，如果你之前有了解过Rails的话，那么本书中的一些概念你会更容易理解。

在本书的大部分章节中，我们会使用到一些Ruby的小魔法，别担心，每个章节我们都会有详细的解释，并且这些小魔法其实也并不是那么难理解，它们只是会让你觉得惊讶：“原来Ruby能让我们这么干！”
