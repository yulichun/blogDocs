## Loki: file size too small\nerror creating index client

解决: 删除loki的持久化目录下的boltdb-shipper-active/index_18xxx目录

参考: https://github.com/grafana/loki/issues/3219

## promtail: context deadline exceeded

```
msg="error sending batch, will retry" status=-1 error="Post \"http://10.0.109.21:3100/loki/api/v1/push\":context deadline exceeded"
```

原因: promtail无法连接loki所致

## promtail cpu使用过高

原因: 由于集群中存在大量的job类pod，这会对loki的服务发现会有很大的压力，需要调整promtail的配置，查看官方的issue，后续可能会将ds由promtail转到服务端来做，promtail需要调整的配置主要为

```yaml
target_config:
sync_period: 30s
positions:
filename: /run/promtail/positions.yaml
sync_period: 30s
```

将 sync_period由默认的10s换成30s

可以使用以下的命令获取到pprof文件分析性能

curl localhost:3100/debug/pprof/profile\?seconds\=20
参考: https://github.com/grafana/loki/issues/1315

## Maximum active stream limit exceeded

原因： 同下，需要调整limit config中的max_streams_per_user， 设置为0即可

## server returned HTTP status 429 Too Many Requests

原因: limit config中的参数: ingestion_burst_size_mb默认值太小，调整即可
```yaml
config:
  limits_config:
    ingestion_rate_strategy: local
    # 每个用户每秒的采样率限制
    ingestion_rate_mb: 15
    # 每个用户允许的采样突发大小
    ingestion_burst_size_mb: 20
```

参考: https://github.com/grafana/loki/issues/1923

## Please verify permissions

原因: 这条其实是warn,不影响promtail的正常工作，如果调整过日志的路径的话要确认promtail挂载的路径是否正常

## loki: invalid schema config

原因: loki的配置文件格式错误.

## promtail: too many open files

原因: /var/log/pods下面的文件数量太多，导致超过内核参数(fs.inotify.max_user_instances)设置配置.

解决：

1. 先查看当前机器设置的配置
cat /proc/sys/fs/inotify/max_user_instances
2. 再查看promtail启动时watch的文件数
cat /run/promtail/positions.yaml | wc -l
3. 如果这个值比max_user_instances要大，则会出现上面的错误，可以通过修改内核参数进行调整
sysctl -w fs.inotify.max_user_instances=1024
4. 生效
sysctl -p
参考: https://github.com/grafana/loki/issues/1153

## promtail: no such file ro directory

原因： promtail daemonset启动时会自动挂载好几个hostpath,如果docker containers的配置调整过，则需要volume跟volumemount都需要对应上.
所以docker挂载点千万不要动。

## grafana获取loki数据报错：too many outstanding requests

出现问题场景：有一个dashboard展示loki内容，有多个loki表达式。将dashboard时间区间设置为最近30m、1h、2h的数据都没问题，但是设置为昨天、最近2天就会出现以上报错。

原因：具体原因不知道，官方网站建议将参数max_outstanding_per_tenant改为2048

最终解决：
问题不仅仅是max_outstanding_per_tenant修改为2048就能解决的，最终是改了多个配置,加入以下配置才得以解决：
```properties
frontend:
  # Maximum number of outstanding requests per tenant per frontend; requests
  #   # beyond this error with HTTP 429.
  #     # CLI flag: -querier.max-outstanding-requests-per-tenant
  #     default 100
  max_outstanding_per_tenant: 2048
  compress_responses: true
  log_queries_longer_than: 20s

frontend_worker:
  grpc_client_config:
  # The maximum size in bytes the client can send.
  #     # CLI flag: -<prefix>.grpc-max-send-msg-size
    max_send_msg_size: 33554432 # 32MiB, default = 16777216]
    max_recv_msg_size: 33554432

ingester_client:
  grpc_client_config:
  # The maximum size in bytes the client can send.
  #     # CLI flag: -<prefix>.grpc-max-send-msg-size
    max_send_msg_size: 33554432 # 32mb, default = 16777216]
    max_recv_msg_size: 33554432

query_scheduler:
  max_outstanding_requests_per_tenant: 2048
  grpc_client_config:
  # The maximum size in bytes the client can send.
  #     # CLI flag: -<prefix>.grpc-max-send-msg-size
    max_send_msg_size: 33554432 # 32mb, default = 16777216]
    max_recv_msg_size: 33554432
```

## grafana获取loki数据报错：maximum of series(500) reached for a single query

原因：max_query_series默认为500

最终解决：修改max_query_series: 100000配置
```properties
limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  max_query_series: 100000 #
  max_query_parallelism: 2 

```