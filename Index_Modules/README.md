# 索引模块 (Index Modules)

**索引模块(Index Modules)**
是为每个索引创建的模块，用于控制与索引相关的所有方面。

## 一、索引设置 (Index Settings)

可以为每个索引设置索引级别。可选值为：

### *static——静态*

它们只能在索引创建时，或在[封闭索引（closed index）](#)上设置。

### *dynamic——动态*

可以使用[update-index-settings API](#)在实时索引(live index)上对其进行更改。

---

**注意**

更改封闭索引（closed index）上的静态或动态索引设置可能会导致设置错误，如果不删除并重新创建索引，则无法纠正这些设置。

---

## 二、静态索引设置

以下是与任何特定索引模块都不相关的所有*静态索引*设置的列表：

### 1. index.number_of_shards

索引应具有的主要分片数。默认为1。此设置只能在创建索引时设置。不能在封闭索引上更改它。

---

**NOTE**

每个索引分片的数量限制**1024**以下。此限制是一个安全限制，可防止意外创建索引，该索引由于资源分配而可能破坏群集的稳定性。可以通过`export ES_JAVA_OPTS="-Des.index.max_number_of_shards=128"`在群集的每个节点上指定系统属性来修改该限制。

---

### 2. index.number_of_routing_shards

用于[拆分](#)索引的路由分片的数量。

例如，`number_of_routing_shards`设置为`30`（5 x 2 x 3）可以除以2或3。换句话说，可以将其拆分成如下：

- 5 -> 10 -> 30 (按2分割，再按3分割)
- 5 -> 15 -> 30 (按3分割，再按2分割)
- 5 -> 30 (按6分割)

此设置的默认值取决于索引中主分片的数量。默认值旨在允许您以2的因子进行拆分，最多可分割1024个碎片。

### 3. index.shard.check_on_startup

打开前是否应检查分片是否损坏。当检测到损坏时，它将防止分片被打开。可选值：

- false（默认）

打开分片时不检查是否损坏。

- checksum

检查物理损坏。

- true

检查物理和逻辑损坏。就CPU和内存使用而言，这要昂贵得多。

---

**WARNING**

仅专家。在大索引上检查分片可能会花费很多时间。

---

### 4. index.codec

默认使用`LZ4`压缩来压缩存储的数据，但是可以将其设置为`best_compression`使用[DEFLATE](#)以获得更高的压缩率，但是会降低存储字段的性能。

如果您正在更新压缩类型，则合并段片后将应用新的压缩类型。可以使用[强制合并](#)来强制段片[合并](#)。

### 5. index.routing_partition_size

自定义[路由](#)值可以到达的分片数量。默认为1，并且只能在创建索引时设置。除非`index.number_of_shards`的值也是1，否则`index.routing_partition_size`的值必须小于`index.number_of_shards`的值。

有关如何使用此设置的更多详细信息，请参见[路由到索引分区](#)。

### 6. index.soft_deletes.enabled

在7.6中已禁用。并将在以后的Elasticsearch版本中删除。不做翻译。

指示是否在索引上启用软删除。软删除只能在创建索引时配置，并且只能在Elasticsearch 6.5.0或之后创建的索引上配置。默认为true。

### 7. index.soft_deletes.retention_lease.period

在认为过期之前保留分片历史保留租约的最大期限。分片历史保留租约可确保在合并Lucene索引期间保留软删除。默认为`12h`。

### 8. index.load_fixed_bitset_filters_eagerly

指示是否为嵌套查询预加载[缓存的过滤器](#)。可选值为true（默认）和false。

## 三、动态索引设置

以下是与任何特定索引模块都不相关的所有*动态索引*设置的列表：

### 1. index.hidden

指示默认情况下是否应隐藏索引。使用通配符表达式时，默认情况下不返回隐藏索引。通过使用`expand_wildcards`参数，可以按请求控制此行为。可选值为true和false（默认值）。

### 2. index.number_of_replicas

每个主分片具有的副本数。默认为1。

### 3. index.auto_expand_replicas

根据集群中数据节点的数量自动扩展副本的数量。设置为以短划线（-，减号）分隔的上下限（例如0-5）或all 用于上限（例如0-all）。默认为false（即禁用）。

请注意，副本的自动扩展数量仅考虑了[分配过滤](#)规则，而忽略了其他任何分配规则，例如[分片分配意识](#)和[每个节点的总分片](#)，如果适用规则阻止了所有复制，这可能导致群集运行状况为`YELLOW`。

### 4. index.search.idle.after

分片在被视为搜索空闲之前不能接收搜索或获取请求的时间。（默认为30s）

### 5. index.refresh_interval

多久执行一次刷新操作，使针对索引做的最近修改对搜索可见。默认为`1s`。可以设置为`-1`禁用刷新。

如果未明确设置此配置，分片在至少`index.refresh_interval`秒内仍未看到请求流量，则分片将不会接收后台刷新（操作）直到分片收到了一个搜索请求。

命中有空闲分片（其中有待刷新）的搜索将等待下一次后台刷新（1s内）。此行为旨在不执行搜索的情况下自动优化默认情况下的批量索引。为了防止此行为，应显示地将刷新间隔设置为1s。

### 6. index.max_result_window

`from + size`搜索到该索引的最大值。默认为`10000`。搜索请求占用的堆内存和时间与`from + size`成正比，因此会限制该内存。有关提高此效果的更有效的选择，请参见[滚动](#)或[搜索之后](#)。

### 7. index.max_inner_result_window

`from + size`内部匹配定义(inner hits definition)和顶部匹配聚合(top hits aggregations)的最大值。默认为`100`。内部命中和顶部命中聚合占用的堆内存和时间与`from + size`成正比，因此会限制该内存。

### 8. index.max_rescore_window

搜索此索引时`rescore`请求的`window_size`的最大值。默认`index.max_result_window`值为`10000`。搜索请求占用的堆内存和时间与`max(window_size, from + size)`成正比，因此会限制该内存。

### 9. index.max_docvalue_fields_search

`docvalue_fields`查询中允许的最大数量。默认为`100`。Doc-Value字段的成本很高，因为它们可能会逐字段查找每个文档。

### 10. index.max_script_fields

`script_fields`查询中允许的最大数量。默认为`32`。

### 11. index.max_ngram_diff

`NGramTokenizer`和`NGramTokenFilter`的`min_gram`和`max_gram`之间的最大允许差异。默认为1。

### 12. index.max_shingle_diff

[shingle令牌过滤器](#)的`max_shingle_size`和`min_shingle_size`之间允许的最大差异。默认为3。

### 13. index.max_refresh_listeners

索引的每个分片上可用的最大刷新侦听器数。这些侦听器用于实现[refresh=wait_for](#)。

### 14. index.analyze.max_token_count

使用_analyze API可以产生的最大令牌数。默认为10000。

### 15. index.highlight.max_analyzed_offset

高亮显示请求将分析的最大字符数。仅当在文本(text)没有使用偏移量(offsets)和术语向量(term vectors)索引时，此设置才适用。默认为1000000。

### 16. index.max_terms_count

术语查询(Terms Query)中可以使用的最大术语数。默认为65536。

### 17. index.max_regex_length

可在Regexp查询中使用的正则表达式的最大长度。默认为1000。

### 18. index.routing.allocation.enable

控制此索引的分片分配。可以设置为：

- `all`(默认) - 允许为所有分片分配分片。
- `primaries` - 仅允许为主要分片分配分片。
- `new_primaries` - 仅允许为新创建的主分片分配分片。
- `none` - 不允许分片分配。

### 19. index.routing.rebalance.enable

为该索引启用分片重新平衡。可以设置为：

- `all`(默认) - 允许所有分片重新平衡。
- `primaries` - 仅允许对主要分片进行分片重新平衡。
- `replicas` - 仅允许对副本分片进行分片重新平衡。
- `none` - 不允许分片重新平衡。

### 20. index.gc_deletes

[删除的文档的版本号](#)可用于[进一步的版本控制操作](#)的时间长度。默认为60s。

### 21. index.default_pipeline

此索引的默认[摄取节点(ingest node)](#)管道。如果设置了默认管道并且该管道不存在，则索引请求将失败。可以使用`pipeline`参数覆盖默认值。特殊管道名称`_none`表示不应运行任何摄取管道。

### 22. index.final_pipeline

此索引的最终[摄取节点](#)管道。如果设置了最终管道并且该管道不存在，则索引请求将失败。最终管道始终在请求管道（如果指定）和默认管道（如果存在）之后运行。特殊管道名称`_none`表示不会运行任何摄取管道。

## 四、其他索引模块中的设置

其他在索引模块中可用的索引设置：

### 1. [Analysis](#)

用于定义分析器，令牌生成器(tokenizers)，令牌过滤器(token filters)和字符过滤器(character filters)的设置。

### 2. [索引分片分配(Index shard allocation)](#)

控制在何处，何时以及如何将分片分配给节点。

### 3. [Mapping](#)

启用或禁用索引的动态映射。

### 4. [Merging](#)

控制分片如何通过后台合并进程合并。

### 5. [相似之处(Similarities)](#)

配置自定义相似性设置以自定义如何对搜索结果进行评分。

### 6. [慢日志(Slowlog)](#)

控制如何记录慢查询和获取请求。

### 7. [Store](#)

配置用于访问分片数据的文件系统的类型。

### 8. [Translog](#)

控制事务日志和后台刷新操作。

### 9. [历史记录保留(History retention)](#)

控制索引中操作历史记录的保留。

### 10. [索引压力(Indexing pressure)](#)

Configure indexing back pressure limits.

## X-Pack索引设置

### 1. [索引生命周期管理](#)

指定索引的生命周期策略和过渡别名。





