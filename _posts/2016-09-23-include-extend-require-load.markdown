---
layout: post
title: include、extend、require、load的区别
date:  2016-09-23 12:00:00
author: "Albert"
tags:
    - Ruby
    - Rails
---

> “初学Ruby，总是会被一些概念所混淆，比如include、extend的区别，require和load的区别，以及eql?，equal?，===之间的区别，本文会对这些概念做一些总结和区分。”

# include VS extend
- - - 
* include：在一个`class`中使用`include`来Mixin一个模块时，会将该模块中的方法引入到类中供类的实例使用。所以，使用`include`引入的模块中的方法是作为类的实例方法被引入。  
* extend：与`include`恰恰相反，使用`extend`来Mixin一个模块时，模块中的方法均会成为该类的类方法。  
{% highlight ruby %}
module A
  def foo
    "class method"
  end
end

module B
  def bar
    "instance method"
  end
end

#include
class A
    include B
end

A.new.bar #=> "instance method"
A.bar #=> "NoMethodError"

#extend
class A
    extend A
end

A.foo #=> "class method"
{% endhighlight%}
> 总结：`include`与`extend`的最大区别即是被引入的模块的方法前者是实例方法，后者是类方法。
- - - 
# require VS load
- - - 
* require：`require`允许你载入一个library并且会阻止你载入多次，当你使用`require`载入一个library多次时，`require`会返回`false`   
* load：`load`方法基本与`require`一致，不过它不会跟踪要导入的library是否被加载，因此，当重复使用`load`载入同一个library时也会载入成功。   

> 总结：大部分情况下你会使用`require`来代替`load`，但当你每次都需要重新加载时你才会考虑使用`load`，例如模块的状态频繁变化的时候，你会使用`load`进行加载。
- - - 
# eql? VS equal? VS === VS ==
- - - 
* eql?：`eql?`和通常意义上的`相等`一样，如果两个对象的值相等，那么`eql?`就返回`true`。但是需要注意一点就是当子类中修改了`==`方法，应该alias给`eql?`，另外还需要注意些特殊的情况，例如：  
{% highlight ruby%}
1 == 1.0 #=> true
1.eql? 1.0 #=> false
{% endhighlight%}
在整数和小数的比较时，`eql?`与`==`的表现会不一致。
* equal?：比较内存地址相同的对象，即两个对象的`object_id`相等才会返回true。 
{% highlight ruby%}
a = "a"
b = "a"
a.equal? b #=> false

c = :a
d = :d
c.equal? d #=> true
{% endhighlight%}
* ===：用在case语句里会调用的方法。   
* ==：按业务需求覆盖该方法。  

