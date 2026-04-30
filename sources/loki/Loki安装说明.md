## 简介

loki是日志系统，promtail是帮助loki收集日志的采集器，grafana用于展示，它们的关系：
promtail =》 loki =》 grafana

## promtail的安装

### 配置文件

下载配置文件：
```shell
$ wget https://raw.githubusercontent.com/grafana/loki/v1.5.0/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml
```
修改配置文件内容：
```yaml
server:
  http_listen_port: 9080 
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml # 游标记录上一次同步位置
  sync_period: 5s # 5秒钟同步一次


clients:
  - url: http://10.0.80.92:3100/loki/api/v1/push # loki地址，收集日志的接口

# 重要的限制
limits_config:
  

scrape_configs:
- job_name: nginx_access_log # job名称
  static_configs:
  - targets:
      - localhost 
    labels: # 自定义标签，日志存储到loki后会带上这些标签
      job: nginx_access_log
      host: 10.0.249.177
      agent: promtail
      __path__: /var/log//TRP/*.log # 要监听的日志
- job_name: rms
  static_configs:
  - targets:
      - localhost
    labels:
      job: rmslogs
      __path__: /var/log/nginx/RMS.log
```

- [promtail比较全面的中文配置说明](https://jishuin.proginn.com/p/763bfbd5863b)
- 官方文档更全面，但有些细节讲的不清楚

### 运行

```shell
$ docker pull grafana/promtail:latest
$ docker run -v /var/log:/var/log grafana/promtail:1.5.0 -config.file=/mnt/config/promtail-config.yaml
```


## loki的安装

### 配置文件

配置文件下载：
```shell
$ wget https://raw.githubusercontent.com/grafana/loki/v1.5.0/cmd/loki/loki-local-config.yaml -O loki-config.yaml
```

配置文件内容：
```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 39095 #grpc监听端口，默认为9095
  grpc_server_max_recv_msg_size: 15728640  #grpc最大接收消息值，默认4m
  grpc_server_max_send_msg_size: 15728640  #grpc最大发送消息值，默认4m

ingester:
  lifecycler:
    address: 192.168.8.253 #IP地址
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s
  max_transfer_retries: 0
  max_chunk_age: 20m  #一个timeseries块在内存中的最大持续时间。如果timeseries运行的时间超过此时间，则当前块将刷新到存储并创建一个新块

schema_config:
  configs:
    - from: 2018-04-15
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 168h

storage_config:
  boltdb:
    directory: /opt/loki/loki_data/index

  filesystem:
    directory: /opt/loki/loki_data/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  ingestion_rate_mb: 30  #修改每用户摄入速率限制，即每秒样本量，默认值为4M
  ingestion_burst_size_mb: 15  #修改每用户摄入速率限制，即每秒样本量，默认值为6M

chunk_store_config:
  max_look_back_period: 168h   #回看日志行的最大时间，只适用于即时日志

table_manager:
  retention_deletes_enabled: true #日志保留周期开关，默认为false
  retention_period: 168h  #日志保留周期

```

-[官方配置说明](https://grafana.com/docs/loki/latest/configuration/)

### 运行

```shell
$ docker pull grafana/loki:1.5.0
$ docker run -p 3100:3100 grafana/loki:1.5.0 -config.file=/mnt/config/loki-config.yaml
```