## 配置优化

我们对influxDB的性能优化进行参数优化，代码层面不涉及。

测试与原理结论可参考该isuue https://dev.koal.com/issues/134991

```yaml
[data]
  dir = "/var/lib/influxdb/data"
  engine = "tsm1"
  wal-dir = "/var/lib/influxdb/wal"
  #wal日志落盘周期，官方建议0-100ms
  #尝试了100ms,50ms,20ms之后，目前折中采用50ms
  wal-fsync-delay = "50ms"
  # 使用inmem索引，经过测试使用inmem内存较为稳定
  index-version = "inmem"
  # 分片允许最大内存,当超过最大内存会拒绝写入
  # 内存越大，多个新老分片会占用更多的堆空间
  cache-max-memory-size = "1g"
  # 用于设置快照大小，大于该值时数据会刷新到tsm文件，默认值：25MB
  cache-snapshot-memory-size = "50m"
  # tsm1引擎快照写盘延迟，当cache超过该延迟时间未写入磁盘，就会将数据写入磁盘，默认值：10m
  cache-snapshot-write-cold-duration = "10m"
  #描述: 使用全量策略压缩冷冻期分片的周期，默认4h
  #场景: 当shard进入冷冻期后，会经过4h,开始进行全量压缩策略，进一步减少shard落盘数据占用的空间
  #与cache-snapshot-write-cold-duration配合使用，可以从日志中看到，新分片开始写入数据之后，在4h+10m之后，会对上个分片进行全量压缩策略
  compact-full-write-cold-duration = "72h"
  # 允许TSM引擎压缩每秒最大落盘数据量，默认值：48m
  # 适量增大可减轻io压力，在大数据量写入时，适当增加该值可以增大吞吐量，如果减小会增大压缩时长
  compact-throughput = "128m"
  # 允许TSM引擎压缩每秒最大落盘数据量,短脉冲串允许以可能更大的值发生，默认值：48m
  # 适量增大可减轻io压力，在大数据量写入时，适当增加该值可以增大吞吐量，如果减小会增大压缩时长
  compact-throughput-burst = "128m"
  # wal日志超过128m时会被压缩为索引文件，并删除，默认1m
  # 如果series（measuremet+tags+fields）和shard时间分片多的话，该值可以设置的小一点，一般取值在64k到1m
  max-index-log-file-size = "256k"
  
[http]
  enabled = true
  flux-enabled = true
  bind-address = "0.0.0.0:64300"

[subscriber]
  enabled = true
  http-timeout = "60s"
  # 重要参数，适量的设置大一点，可以增加入库效率
  write-concurrency = 80

[[udp]]
  # High-traffic UDP
  enabled = true
  bind-address = ":64600" # the bind address
  database = "telegraf" # Name of the database that will be written to
  # 重要参数，适量的设置大一点，可以增加入库效率
  batch-size = 5000 # will flush if this many points get buffered
  batch-timeout = "2s" # will flush at least this often even if the batch-size is not reached
  # 重要参数，不能设置太大，避免占用过多内存，在压测情况下会吃很多内存
  batch-pending = 5 # number of batches that may be pending in memory
  # 根据业务情况适当设置，一般默认就行
  read-buffer = 8388608 # (8*1024*1024) UDP read buffer size

```

## 其他注意事项

- rentention policy的shard group duration不要设置太小，可以使用默认值；
- series cardinality 不要太大；
  - measurement和tag不要异常的多；
  - tag的值不能很多，如果一个字段的值很多，千万不要把这个字段作为tag；