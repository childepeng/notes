# Flink

Flink 是一个分布式的数据处理引擎，可对有界数据或者无界数据进行有状态或者无状态计算。

**有界流**：有定义数据的开始和结束，有界流可以在接收所有数据后再进行计算；有界流所有数据可以被排序；有界流处理通常被称为批处理。

**无界流**：有定义流的开始，没有定义流的结束；数据持续不断的产生。

## Flink程序组成Source、Transformation、Sink





## Flink数据分发策略

Flink中算子可以设置不同的并行度，由此就产生一个问题，在算子多并行的场景下，上游算子的输出和下游算子的输入是如何分发的呢？

Flink提供了几种数据分发策略：

### GlobalPartitioner

上游算子的数据都发送到下游 0 号分区

### ShutfflePartitioner

上游算子的数据随机分发到下游分区

### RebalancePartitioner

上游算子数据循环依次发送到下游分区

### RescalePartitioner

就近原则；

### BroadcastPartitioner

下游每个算子都会收到全部数据

### ForwardPartitioner

上下游算子分区需要相同，数据采用一对一的方式分发