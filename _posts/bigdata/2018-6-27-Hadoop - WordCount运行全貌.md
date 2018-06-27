---
layout: post
title: 大数据学习-WordCount运行全貌
date: 2018-6-26
excerpt: "大数据学习-WordCount运行全貌"
categories: 大数据
tags: [大数据]
---


# 简介

介绍WordCount运行全貌，探讨mapper和reducer之间的关系，Hadoop的内部工作机制。

# 1. 启动

调用驱动中的Job.waitForCompletion()是所有行动的开始。

- 该驱动是程序唯一一段运行在本地机器上的代码
- 开启了本地主机和JobTracker通信。
- JobTracker负责作业调度和执行的各个方面，是执行如何与作业管理相关的任务时的主要连接口
- JobTracker代表我们与NameNode通信，并对存储在HDFS上的数据相关的所有交互进行管理。

# 2. 输入分块

HDFS通常被分成至少64MB的数据块，JobTracker会将每个数据块分配给一个map任务。

- 这些交互发生在JobTracker接受输入数据，并确定如何将其分配给map任务的时候
- 各分块完成了运算，JobTracker就会将它们和包含mapper与reducer类的jar文件放置在HDFS上作业专用的目录，而该路径在任务开始时将被传递给每个任务

# 3. 任务分配

JobTracker确定了所需的map任务数，它就会检查集群中的主机数，正在运行的TaskTracker数以及可并发执行的map任务数（用户自定义的配置变量）

- JobTracker会尝试定义一个执行计划，使TaskTracker尽可能处理位于相同物理主机上的数据块。
- 或者即使做不到这点，TaskTracker至少处理一个位于相同硬件机架中的数据块
- 数据局部性优化是Hadoop能高效处理巨大数据集的一个关键原因。
- 默认情况下，每个数据块被复制到三台不同的主机
- 本地处理大部分数据块的任务/主机计划比起初预想的可能性更高。

# 4. 启动任务

每个TaskTracker开启一个独立Java虚拟机来执行任务。

- 虽然增加了启动时间损失，但它可以隔离运行map或reducer所引发的问题和TaskTracker
- 而且可以配置成在随后的任务之间共享
- 如果集群有足够的能力一次性执行所有的任务，它们将会被全部启动，并获得它们将要处理的分块数据和作业jar文件 
- 每个TaskTracker随后将分块复制到本地文件系统
- 如果任务数超过了集群能力，JobTracker将维护一个挂起任务队列，并在节点完成最初分配的map任务后，将挂起任务分配给节点

# 5. 不断监视JobTracker

JobTracker等待TaskTracker执行所有的mapper和reducer。不断地与TaskTracker交换心跳和状态下次，查找进度或问题的证据。

JobTracker从整个作业执行过程的所有任务中收集指标，其中一些指标是Hadoop提供的，还有一些是map和reduce任务的开发人员制定。

# 6. mapper的输入

WordCount实例驱动类使用TextInputFormat指定了输入文件的格式和结构。因此Hadoop会把输入文件看做以行号为键并以改行内容为值的文本

# 7. mapper的执行

依据作业的配置方式，mapper接收到的键/值对分别是相应行在文件中的偏移量以及该行内容。

- WordCount实例类的mapper方法舍弃了键并使用标准的java string类的split方法将每行文本内容拆分成词（使用正则表达式或StringTokenizer可以更好的短词）
- 针对每个单独的词，mapper输出由单词本身组成的键和值

# 8. mapper的输出和reducer的输入

mapper的输出是一系列形式为（word,1）的键值对

- 键值对并不会直接传给reducer，中间还有一个shuffle阶段，这也是许多MapReduce奇迹发生的地方

# 9. 分块

Reduce接口的隐性保证之一是，与给定键值相关的所有值都会被提交到同一个reducer。

- 一个集群中运行着多个reduce任务，每个mapper的输出必须被分块，使其分别传入相应的各个reducer。这些分块文件保存在本地节点的文件系统
- 集群中的reduce任务数并不像mapper数量一样是动态的，我们可以在作业提交阶段指定reduce任务数。因此每个TaskTracker就知道集群中有多少个reduce，并据此得知mapper输出应切分为多少块

# 10. 可选分块函数

默认情况下，Hadoop将对输出的键进行哈希运算，从而实现分块。

- 功能有org.apache.hadoop.mapreduce.lib.artition包中的HashPartitioner类实现。
- 用户可以自定义Partitioner子类，实现针对具体应用的分块逻辑。特别是标准哈希导致数据分布不均匀时。

# 11. reducer类的输出

reducer的TaskTracker从JobTracker接收更新，指明集群中那些节点承载着map的输出分块。

之后TaskTracker从各个节点获取分块，并将它们合并成一个文件反馈给reducer任务处理

# 12. reducer类的执行

WordCountReducer类很简单，针对每个词实现词频统计。最后输出(word, count)

- 通常reducer的调用次数小于mapper的调用次数。如果有非常严格的性能要求，可以采用任何有利于提高性能的措施
- 先期MapReduce对数据集执行标准化或清理策略

# 13. reducer类的输出

WordCount最后输出是(word, count)，这些数据将被输出到驱动程序指定的输出路径下的分块文件中，并将使用指定的OutputFormat对其进行格式化。

- 每个reducer任务写入一个以part-r-nnnnn为文件名的文件，其中nnnnn从00000开始并逐步递增。

# 14. 关机

一旦成功完成所有任务，JobTracker向客户端输出作业的最终状态，以及作业运行过程中一些比较重要的计数器集合。

- 完整的作业和任务历史记录存储在每个节点的日志路径中，通过JobTracker节点的50030端口可以访问


# 小结

流程图

![](https://i.imgur.com/mkSA9pM.png)


# Reference


