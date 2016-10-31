---
layout: post
title: Spring Batch 
date:  2016-10-30 22:42:00
author: "Albert"
tags:
    - Java 
    - Spring Batch 
---

> “Spring Batch针对企业级应用中对大批量的数据进行基础的批处理的需求提供了一系列可重用的功能，包括日志/跟踪，事务管理，任务处理统计，任务重启、跳过以及资源管理。并且，它通过优化和分区技术提供了一些额外的更先进的服务和功能，这些功能能使用户处理更大数据量的批处理任务，并且能得到更好的性能保证。不管是简单的还是复杂的大数据量任务，都可以通过Spring Batch这个高度可扩展的框架来进行处理。”

# 基本功能 - Feature  
- - - 
1. 事务管理 - Transaction management  
2. 基于块的任务处理 - Chunk Based Processing  
3. 声明式IO - Declarative I/O  
4. 任务启动/跳过/重启 - Start/Skip/Restart  
5. 任务重试/跳过 - Retry/Skip  
6. 基于web的后台管理接口 - Web Bases administration interface(Spring Batch Admin)  

# 使用场景 - Usage Scenarios  
- - - 

1. 周期性的批处理任务 - Commit batch process periodically  
2. 并发执行的批处理任务 - Concurrent batch processing: parallel processing of a job  
3. 消息驱动的批处理任务 - Staged, enterprise message-driven processing  
4. 高并发量的并行批处理任务 - Massively parallel batch processing  
5. 手动或定时重启失败任务 - Manual or scheduled restart after failure  

# 基本架构 - Spring Batch Architecture  
- - - 

Spring Batch的基本架构可以用下图来表示：  

![Spring Batch layers](/img/spring-batch-layers.png)

这张图简要的描述了`Spring Batch`的三个主要的组成部分，`Application`，`Batch Core`，`Batch Infrastructure`   
`Application`部分包含了`Job`的创建和定义。  
`Core`部分包含了该框架在运行时的一些重要依赖，例如launch and controll a job，`JobLauncher`，`Job`，`Step`等，这三个部分是构成Spring Batch的核心。  
`Infrastructure`部分则包含了一些框架本身以及开发者均会使用到部分，例如`ItemReader`，`ItemWritter`以及一些公共的服务，如`RetryTemplete`。  

# 框架DSL  

![Spring Batch refrence model](/img/spring-batch-reference-model.png)  

上图是一个简化的Spring Batch批处理框架的运行关系图，该图中包含的`JobLauncher`，`Job`，`Step`以及`JobRepository`是Spring Batch中最为核心的组成部分。  
他们之间的关系可以简单地总结如下：  
一个`Job`包含一个或多个`Step`（`Job`是`step`的容器），而每一个`Step`则包含数据的读取（`ItemReader`），数据处理（`ItemProcessor`），数据的写入（`ItemWritter`）。  
一个`Job`需要被执行（launch），需要`JobLauncher`，而当前正在执行的任务的各种元数据（`meta-data`）则需要被存储，此时需要`JobRepository`。  


## Job
`Job`是一个封装了完整批处理过程的实体（可以理解为一个对象）。一个`Job`可以通过一个XML配置文件或者一个`Java Based`配置文件wire到一起，通常将这种配置文件成为`Job Configuration`。  

![job-hierarchy](/img/job-heirarchy.png)  

`Job Configuration`包含了如下几种配置：  
1. `Job`的名称。  
2. `Step`s 的定义以及顺序的制定。  
3. `Job`能否被重新执行。  
例如：  
{% highlight ruby %}
<job id="footballJob">
  <step id="playerload" next="gameLoad"/>
  <step id="gameLoad" next="playerSummarization"/>
  <step id="playerSummarization"/>
</job>
{% endhighlight%}

## JobInstance

`JobInstance`指的是一个可运行的`Job`实例，可以将`JobInstance`理解为`Job`new出来的一个实例，它可以根据不同的`JobParameter`生成不同的`JobInstance`。  

## JobParameters

`JobParameters`是一组用来启动批处理任务的参数，它们可以用来识别不同的`JobInstance`，甚至在运行期间用作参考数据。  

![job-parameters](/img/job-stereotypes-parameters.png)  

`JobInstance`和`Job`之间的关系可以用如下等式来描述：  
`JobInstance` = `Job` + identifying `JobParameters`  

# JobExecution  

`JobExecution`指的是一个`JobInstance`的单个执行过程，需要注意的是，`JobExecution`与`JobInstance`均有两个完成状态，即任务完成和任务未完成，但区别在于，`JobExecution`不管执行成功与否，都会变为完成状态，但如果`JobExecution`执行失败了，对应的`JobInstance`就不能被认为是已完成（completed）的。即该`JobInstance`在下一次执行的时候是另外一个`JobExecution`。

> "A `Job` defines what a job is and how it is to be executed, and `JobInstance` is a purely organizational object to group executions together, primarily to enable correct restart semantics. A `JobExecution`, however, is the primary storage mechanism for what actually happened during a run."

## Step  

## JobRepository  

`JobRepository`，顾名思义，是一个仓库，它是上文中提到的`Job`在执行过程中各种需要被持久化存储的状态的repository，它为`JobLauncher`，`Job`和`Step`的实现提供CRUD操作，当一个`Job`被启动的时候，会从`JobRepository`中获取一个`JobExecution`，并且在执行过程中通过将它们传入到`JobRepository`中实现持久化。  

## JobLauncher  

`JobLauncher`是一个通过接受一组`JobParameters`并启动`Job`的简单接口。  
{% highlight java%}
public interface JobLauncher {
  public JobExecution run(Job job, JobParameters jobParameters)
    throws JobExecutionAlreadyRunningException, JobRestartException;
}
{% endhighlight %}  
该接口期望实现从`JobRepository`获取一个有效的`JobExecution`并执行`Job`。   

## ItemReader  
TODO  
## ItemProcessor  
TODO  
## ItemWritter  
TODO  














