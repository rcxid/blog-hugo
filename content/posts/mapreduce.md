+++
title = "MapReduce"
date = "2025-06-25T18:29:21+08:00"
summary = "mapreduce原理"
categories = ["数据处理"]
tags = ["大数据", "hadoop", "mapreduce"]
draft = false
+++

## 原理
Hadoop的MapReduce可分为MapTask和ReduceTask两个阶段。第一阶段的MapTask之间并行运行，互不影响。第二阶段的ReduceTask同样是并行处理，互不影响，但是依赖第一阶段的MapTask的输出结果。第一阶段的MapTask任务将计算结果先写到内存缓存区，然后溢写到文件，保存到hdfs，这个过程称为Map阶段的shuffle。在数据写入文件时，数据会进行排序和分区，分区的目的是将数据划分到不同的ReduceTask上，下游的ReduceTask是按照partition读取对应的数据，然后进行相应的计算。ReduceTask阶段也存在shuffle，ReduceTask阶段，会将MapTask阶段输出数据，先拉取到内存缓冲区，如果超过阈值，数据同样会被写到hdfs上，这个过程多个文件会进行归并排序，生成一个key全局有序的大文件。这个过程就是ReduceTask阶段的shuffle。shuffle涉及到写hdfs，会影响程序的性能。

## 思考
在Flink流式计算任务中，算子的数据处理，以及算子间的数据传递的这个过程，和Hadoop中的MapReduce大同小异，以Flink的WordCount程序举例，程序调用算子计算顺序大致如下：source.flatMap(...).keyBy(...).sum(...)，flatMap算子对数据的处理，就类似MapReduce的Map阶段。keyBy将flatMap算子计算的数据发送到下游sum算子进行聚合操作，keyBy会将相同的key发送到相同的slot进行计算，这个过程类似MapReduce中Map阶段的shuffle，不同的是上面的Flink将数据存储在内存中。当然Flink程序支持自定义状态计算，并且支持设置状态后端，状态数据可以存储hdfs保存。sum算子会对相同的key的数据进行聚合，这个过程类似MapReduce的Reduce阶段。在Flink开发过程中，优化状态存储占用，能够提升程序的稳定性，比如开窗聚合任务中，采用多级开窗聚合能够大幅度降低状态的大小。
