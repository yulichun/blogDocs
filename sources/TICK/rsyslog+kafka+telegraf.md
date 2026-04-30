# 概述

本文阐述rsyslog->kafka->telegraf的对接配置。
在面对不同的业务需求时，配置也会不同，这里没有描述各个中间件的详细配置方法，只是大致描述各个中间件的配置中重要的环节，具体每个环节的丰富的功能可以参考各自的官网。

# rsyslog
## rsyslog安装

rsyslog对接kafka需要v8.7.0版本及以上。
```
root# apt-get install rsyslog
root# apt-get install rsyslog-kafka
root# rsyslogd -N1
```
## rsyslog配置

这里的场景是rsyslog以syslog方式接收nginx发送来的访问日志，rsyslog需要打开514端口

```
# 接收syslog
module(load="imudp")
input(type="imudp" address="127.0.0.1" port="514")

module(load="imtcp")
input(type="imtcp" address="127.0.0.1" port="514")

# 加入kafka模块
module(load="omkafka")
# 构建要发送给kafka的信息，msg是syslog消息体中的msg字段，也就是访问日志内容
template(name="rawFormat" type="string" string="%msg%")
# 转发到kafka的test_nginx的topic
action(type="omkafka" topic="test_nginx" broker="127.0.0.1:9092" Template="rawFormat")
```

## rsyslog启动

```
root# systemctl restart rsyslog
```

# kafka

使用docker安装kafka，参考：https://hub.docker.com/r/bitnami/kafka。

## kafka安装及启动

注意，这里使用的是最新的kafka的kraft方式部署kafka，用不着zookeeper，docker-compose配置：
```
version: '3'

services:
  kafka:
    image: bitnami/kafka:latest
    container_name: kafka
    network_mode: host
    volumes:
    # kafka配置文件映射
      - ./cfg/server.properties:/bitnami/kafka/config/server.properties
      # kafka的数据持久化目录映射
      - ./data/kafka:/bitnami/kafka/data
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@127.0.0.1:9093
      - ALLOW_PLAINTEXT_LISTENER=yes

```

启动：
```
root# docker-compose up -d kafka
```

## 创建topic

创建名为test_nginx的topick
```
# 进入kafka容器
root# docker exec -it kafka bash
容器# cd /opt/bitnami/kafka/bin
容器# ./kafka-topics.sh --create --replication-factor 1 --partitions 1 --topic test_nginx --bootstrap-server localhost:9092
```

# telegraf

## telegraf配置

```
## 输入
[[inputs.kafka_consumer]]
  ## Kafka brokers.
  brokers = ["127.0.0.1:9092"]

  ## Topics to consume.
  topics = ["test_nginx"]

  ## When set this tag will be added to all metrics with the topic as the value.
 #topic_tag = "msg"

  ## Optional Client id
  # client_id = "Telegraf"

  ## Set the minimal supported Kafka version.  Setting this enables the use of new
  ## Kafka features and APIs.  Must be 0.10.2.0 or greater.
  ##   ex: version = "1.1.0"
  # version = ""

  ## Optional TLS Config
  # tls_ca = "/etc/telegraf/ca.pem"
  # tls_cert = "/etc/telegraf/cert.pem"
  # tls_key = "/etc/telegraf/key.pem"
  ## Use TLS but skip chain & host verification
  # insecure_skip_verify = false

  ## Optional SASL Config
  # sasl_username = "kafka"
  # sasl_password = "secret"

  ## Name of the consumer group.
  # consumer_group = "telegraf_metrics_consumers"

  ## Initial offset position; one of "oldest" or "newest".
  # offset = "oldest"

  ## Consumer group partition assignment strategy; one of "range", "roundrobin" or "sticky".
  # balance_strategy = "range"

  ## Maximum length of a message to consume, in bytes (default 0/unlimited);
  ## larger messages are dropped
  max_message_len = 1000000

  ## Maximum messages to read from the broker that have not been written by an
  ## output.  For best throughput set based on the number of metrics within
  ## each message and the size of the output's metric_batch_size.
  ##
  ## For example, if each message from the queue contains 10 metrics and the
  ## output metric_batch_size is 1000, setting this to 100 will ensure that a
  ## full batch is collected and the write is triggered immediately without
  ## waiting until the next flush_interval.
  max_undelivered_messages = 2000
  data_format = "logfmt"
  name_override = "test_nginx"
  tag_keys = ["app_id"]
  [inputs.kafka_consumer.tags]
    test_nginx = "user_access"

## 输出到loki
[[outputs.loki]]
domain = "http://10.0.230.211:3100"
namepass = ["test_nginx"]

```
重启telegraf即可