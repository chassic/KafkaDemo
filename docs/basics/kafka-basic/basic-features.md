## Kafka特性

### 怎样实现高可用？

- 常见的做法是将不同的 Broker 分散运行在不同的机器上
- 备份机制（Replication）
- 重平衡（Rebalance）
  因为由重平衡引发的消费者问题

### 怎样实现伸缩性？

- 分区（Partitioning）

  类比 MongoDB 和 Elasticsearch 中的 Sharding、HBase 中的 Region



#### 怎样实现吞吐量？

- 消费者组（Consumer Group）