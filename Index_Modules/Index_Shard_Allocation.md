# Index Shard Allocation(索引分片分配)

此模块为每个索引提供设置，以控制分片到节点的分配：

- [分片分配过滤(Shard allocation filtering)](#)
：控制将哪些分片分配给哪些节点。
- [延迟分配(Delayed allocation)](#)
：由于节点离开而导致未分配碎片的延迟分配。
- [每个节点的总分片数](#)
：对每个节点相同索引中的分片数进行强制限制。
