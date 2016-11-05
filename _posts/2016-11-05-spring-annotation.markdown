---
layout: post
title: Spring3.0-基于annotation的依赖注入实现 
date:  2016-11-05 22:42:00
author: "Albert"
tags:
    - Java 
    - Spring 
    - Annotation 
---
> “Spring的依赖配置方式与Spring框架的内核自身是`低耦合`设计的。然后，直到Spring3.0以前，使用XML进行依赖配置几乎是唯一的选择。Spring3.0的出现改变了这一状况，它提供了一系列的针对依赖注入的注解（Annotation）,这使得Spring IoC在XML文件之外多了一种可行的选择。本文将详细介绍如何使用这些注解进行依赖配置的管理。”  

- - -  

# 使用@Repository、@Service、@Controller和@Component将类标识为Bean  

Spring自2.0版本开始，陆续引入了一些注解用于简化Spring的开发。@Repository注解便是属于最先引入的一批，它用于将数据访问层（DAO层）的类标识为Spring Bean。具体只需将该注解标注在DAO类上即可。同时，为了让Spring能够扫描类路径中的类并识别出@Repositoty注解，需要在XML配置文件中启用Bean的自动扫描功能，这可以通过`<context:component-scan/>`实现。如下所示：
{% highlight java%}
// 首先使用 @Repository 将 DAO 类声明为 Bean 
package bookstore.dao; 
@Repository 
public class UserDaoImpl implements UserDao{ …… } 

// 其次，在 XML 配置文件中启动 Spring 的自动扫描功能  
<beans … > 
……
<context:component-scan base-package=”bookstore.dao” /> 
……
</beans>
{% endhighlight%}
如此，我们就不在需要在XML中显式的使用<bean/>进行Bean的配置。Spring在容器初始化的时候将自动扫描base-package指定的包及其子包下的所有文件中的class，所有标注了@Repository的类都将被注册为Spring Bean。  

为什么@Repository只能标注在DAO类上呢？这是因为该注解的作用不只是将类识别为Bean，同时它还能将所标注的类中跑出的数据访问异常封装为Spring的数据访问异常类型。Spring本身提供了一个丰富的并且是与具体的数据访问技术无关的数据访问异常结构，用于封装不同持久层框架抛出的异常，使得异常独立于底层的框架。  

Sping2.5在@Repository的基础上增加了功能类似的额外三个注解：`@Component`、`@Service`、`@Controller`，它们分别用于软件系统的不同层次：  

- `@Component`是一个泛化的概念，仅仅标识一个组件（Bean），可以作用在任何层次。  
- `@Service`通常作用在业务层，但是目前该功能与@Component相同。  
- `Controller`通常作用在控制层，但是目前该功能与@Component相同。 

通过在类上使用使用@Repository、@Component、@Service和@Controller注解，Spring会自动创建相应的BeanDefinition对象，并注册到ApplicationContext中。这些类就成了Spring受管组件。这三个注解除了作用于不同软件层次的类，其使用方式与@Repository是完全相同的。  

另外，除了上面的四个注解外，用户可以创建自定义的注解，然后在注解上标注@Component，那么，该自定义注解便具有了与@Component相同的功能，不过这个功能并不常用。

当一个Bean被自动检测到时，会根据那个扫描器的BeanNameGenerator策略生成它的bean名称。默认情况下，对于包含了name属性的@Component、@Repository、@Service、@Controller，会把name取值作为Bean的名字，如果这个注解不包含name值或是其他被自定义过滤器发现的组件，默认Bean名称会是小写开头的非限定类名，如果你不想使用默认bean命名策略，可以提供一个自定义的命名策略。首先实现BeanNameGenerator接口，确认包含了一个默认的无参数构造方法。然后在配置扫描器时提供一个全限定类名，如下所示：  

{% highlight java%}
<beans ...> 
<context:component-scan 
base-package="a.b" name-generator="a.SimpleNameGenerator"/> 
</beans>
{% endhighlight%}

与通过XML配置的Spring Bean一样，通过上述注解标识的Bean，其默认作用于是`Singleton`，为了配合这四个注解，在标注Bean的同时能够指定Bean的作用域，Spring2.5引入了@Scope注解。使用该注解时只需提供作用域的名称就行了，如下所示： 
{% highlight java%}
@Scope("prototype") 
@Repository 
public class Demo { … }
{% endhighlight%}

如果你想提供一个自定义的作用域解析策略而不使用基于注解的方法，只需实现ScopeMetadataResolver接口，确认包含一个默认的没有参数的构造方法。然后在配置扫描器时提供全限定类名： 

{% highlight java%}
<context:component-scan base-package="a.b"
scope-resolver="footmark.SimpleScopeResolver" />
{% endhighlight%}

# 使用@PostConstruct和@PreDestory指定生命周期回调方法 

Spring Bean是受Spring IoC容器管理的，由容器进行初始化和销毁的（prototype类型由容器初始化之后便不受容器管理），通常我们不需要关注对Bean的初始化和销毁操作，由Spring经过构造函数或者工厂方法创建Bean就是已经初始化完成并立即可用的。然而在某些情况下，可能需要我们手工做一些额外的初始化或者销毁操作，这通常是针对一些资源的获取和释放操作。Spring 1.x为此提供了两种方式供用户指定执行生命周期回调的方法。

第一种方式是实现Spring提供的两个接口：`InitializingBean`和`DisposableBean`。如果希望在Bean初始化完成之后执行一些自定义操作，则可以让Bean实现InitializingBean接口，该接口包含一个afterPropertiesSet()方法，容器在为该Bean设置了属性之后，将自动调用该方法；如果Bean实现了DisposableBean接口，则容器在在销毁该Bean之前，将调用该接口的destory()方法。这种方式的缺点是，让Bean类实现Spring提供的接口，增加了代码与Spring框架的耦合度，因此不推荐使用。 

第二种方式是在XML文件中使用`<bean>`的init-method和destory-method属性指定初始化之后和销毁之前的回调方法，代码无需实现任何接口。这两个属性的取值是相应Bean类中的初始化和销毁方法，方法名任意，但是方法不能有参数。示例如下： 

{% highlight java%}
<bean id=”userService” 
class=”bookstore.service.UserService” 
init-method=”init” destroy-method=”destroy”> 
…
</bean>
{% endhighlight%}

Spring 2.5在保留以上两种方式的基础上，提供了对JSR-250的支持。JSR-250规范定义了两个用于指定声明周期方法的注解：
@PostConstruct和@PreDestory。这两个注解使用非常简单，只需分别将它们标注与初始化之后执行的回调方法或者销毁之前执行的回调方法上，由于使用了注解，因此需要配置相应的Bean后处理器，亦即在XML中增加如下一行： 
{% highlight java%}
 <context:annotation-config />
{% endhighlight%}

比较上述三种指定生命周期回调方法的方式，第一种是不建议使用的，不但其用法不如后两种方式灵活，而且无形中增加了代码与框架的耦合度。后面两种方式开发者可以根据使用习惯选择其中一种，但是最好不要混合使用，以免增加维护的难度。  

- - - 

# 使用@Required进行Bean的依赖检查 

依赖检查的作用是，判断给定Bean的相应`Setter`方法是否都在实例化的时候被调用了，而不是判断该Bean的实例中某个字段是否已经存在值。Spring进行依赖检查时，只会判断属性是否使用了Setter注入。如果某个属性没有使用Setter注入，即使是通过构造方法为该属性注入了值，Spring依然认为他没有执行注入，从而抛出异常。另外，Spring只管是否通过Setter执行了注入，而对注入的值却没有任何要求，即使注入的是`null`，Spring也认为是执行了依赖注入。  
`<bean>` 标签提供了 dependency-check 属性用于进行依赖检查。该属性的取值包括以下几种： 
- none -- 默认不执行依赖检查。可以在 `<beans>`标签上使用 default-dependency-check 属性改变默认值。 
- simple -- 对原始基本类型和集合类型进行检查。 
- objects -- 对复杂类型进行检查（除了 simple 所检查类型之外的其他类型）。 
- all -- 对所有类型进行检查。 

旧版本使用 dependency-check 在配置文件中设置，缺点是粒度较粗。使用 Spring2.0 提供的 @Required 注解，提供了更细粒度的控制。@Required 注解只能标注在 Setter 方法之上。因为依赖注入的本质是检查 Setter 方法是否被调用了，而不是真的去检查属性是否赋值了以及赋了什么样的值。如果将该注解标注在非 setXxxx() 类型的方法则被忽略。为了让 Spring 能够处理该注解，需要激活相应的 Bean 后处理器。要激活该后处理器，只需在 XML 中增加如下一行即可。 

{% highlight java%}
<context: annotation-config/>
{% endhighlight%}

- - - 

当某个被标注了@Required的Setter方法没有被调用，则Spring在解析的时候会抛出异常，以提醒开发者对相应属性进行设置。 
- - - 

# 使用 @Resource、@Autowired 和 @Qualifier 指定 Bean 的自动装配策略 






