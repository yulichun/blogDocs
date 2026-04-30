## loki

loki是一款类prometheus的日志系统，基于这样的概念实现：以标签来索引日志。日志存储分两块内容，一块内容是日志的标签，根据标签可以索引到具体的日志；另一块是日志内容，压缩存储在块中，可以用S3、GCS存储系统，甚至可以用文件系统。

loki支持的功能有：
- 日志查询操作，支持聚合、过滤，以及模板函数
- 支持集成告警系统，如alertManager
- 支持http api
- 支持分布式，可以拆分成多个组件分布式部署

下面重点介绍查询表达式logQL

## logQL

loki提供了丰富的日志查询表达式，在官网有详细的介绍，这里只提供主要功能的描述

### 基本操作符

#### 数值操作（Arithmetic operators）

- + (加)
- - (减)
- * (乘)
- / (除)
- % (余)
- ^ (power/exponentiation)

#### 关系操作（Logical and set operators）

- and (intersection)
- or (union)
- unless (complement)

#### 比较操作（Comparison operators）

- == (equality)
- != (inequality)
- \> (greater than)
- \>= (greater than or equal to)
- < (less than)
- <= (less than or equal to)

### log query-重点

log query操作简单来说就是查询日志的行内容。目标是日志中的每一行内容，可以根据表达式去查询过滤行，也可以利用表达式重新格式化要展示的行。

#### 日志流选择器（log stream sellector）

最基本的日志流选择器语句，{}内包含两个标签：
```
{app="mysql",name="mysql-backup"}
```
标签相关的操作符（label matching operator）
- =: 等于，如上面app标签等于mysql
- !=: 不等于
- =~: 正则匹配
- !~: 正则不匹配

#### 日志管道（Log pipeline）

日志管道操作就类似于linux命令行中的管道操作，“|”号间隔的每段表达式表示管道中一段操作，这每一段操作组成了一个流式的数据处理。

```sql
{cluster="ops-tools1", name="querier", namespace="loki-dev"} // 根据标签匹配日志
  |= "metrics.go" != "loki-canary" // “|”号间隔多端表达式
  | logfmt
  | query != ""
```

日志管道操作有：
- 过滤表达式：用于过滤日志
  - 行过滤表达式（line filter expressions）
  - 标签过滤表达式（label filter expressions）
- 解析表达式（Parsing expressions）：用于解析日志，从日志行中解析出标签
- 格式化表达式：用于格式化要展示的内容
  - 行格式化表达式（line format expressions）
  - 标签格式化表达式（label format expressions）

##### 行过滤表达式（line filter expressions）

- |=: 日志行包含某个字符串
- !=: 日志行不包含某个字符串
- |~: 日志行包含正则可以匹配的字符串内容
- !~: 日志行不包含正则可以匹配的字符串内容

```text
不包含错误信息的日志
{app="nginx",host="localhost"} != error
```

##### 标签过滤表达式（label filter expressions）

- == 或 = 表示等于
- != 表示不等于
- \> 和 >= 表示大于和大于等于
- < and <= 表示小于和小于等于
```text
nginx中host标签是localhost并且method标签为GET的日志
{app="nginx"} | host == "localhost" and method="GET"
```

##### 解析表达式（Parsing expressions）

解析表达式目的是根据日志行解析出该行的标签值

1. json：针对日志是json格式的一种解析方式,其有无参和有参两种解析方式
- 无参解析：
通过|json 可以将下面数据：
```json
{
    "protocol": "HTTP/2.0",
    "servers": ["129.0.1.1","10.2.1.3"],
    "request": {
        "time": "6.032",
        "method": "GET",
        "host": "foo.grafana.net",
        "size": "55",
        "headers": {
          "Accept": "*/*",
          "User-Agent": "curl/7.68.0"
        }
    },
    "response": {
        "status": 401,
        "size": "228",
        "latency_seconds": "6.031"
    }
}
```
解析出的标签：
```text
"protocol" => "HTTP/2.0"
"request_time" => "6.032"
"request_method" => "GET"
"request_host" => "foo.grafana.net"
"request_size" => "55"
"response_status" => "401"
"response_size" => "228"
"response_size" => "228"
```
- 有参解析：
 通过 | json first_server="servers[0]", ua="request.headers[\"User-Agent\"]表达式可以将下面的
```text
{
    "protocol": "HTTP/2.0",
    "servers": ["129.0.1.1","10.2.1.3"],
    "request": {
        "time": "6.032",
        "method": "GET",
        "host": "foo.grafana.net",
        "size": "55",
        "headers": {
          "Accept": "*/*",
          "User-Agent": "curl/7.68.0"
        }
    },
    "response": {
        "status": 401,
        "size": "228",
        "latency_seconds": "6.031"
    }
}
```
解析出标签：
```text
"first_server" => "129.0.1.1"
"ua" => "curl/7.68.0"
```
2. logfmt：针对符合logfmt格式的日志行
通过| logfmt 可以将下面数据：
```text
at=info method=GET path=/ host=grafana.net fwd="124.133.124.161" service=8ms status=200
```
解析出标签：
```text
"at" => "info"
"method" => "GET"
"path" => "/"
"host" => "grafana.net"
"fwd" => "124.133.124.161"
"service" => "8ms"
"status" => "200"
```
3. Pattern
不常用，参考官网
4. 正则匹配（Regular expression）：利用正则表达式解析
通过 | regexp "(?P<method>\\w+) (?P<path>[\\w|/]+) \\((?P<status>\\d+?)\\) (?P<duration>.*)" 可以将
```text
POST /api/prom/api/v1/query_range (200) 1.5s
```
解析出标签：
```text
"method" => "POST"
"path" => "/api/prom/api/v1/query_range"
"status" => "200"
"duration" => "1.5s"
```
5. unpack
类似于json解析，多一个将解析标签结果格式化为日志新内容。

##### 行格式化表达式（line format expressions）

行格式化表达式目的是格式化日志行，通俗点说，就是通过表达式中定义日志行要展示的新内容。

##### 标签格式化表达式（label format expressions）

标签格式化表达式目的是格式化日志行的标签。

### 时序数据查询（metric query）

主要是利用标签查询，包含多种聚合操作。

#### 日志范围聚合（Log range aggregations）

- rate：计算每秒的条目数
- count_over_time：计算给定范围内每个日志流的条目

例子：

```sql
# 三十分钟日志行记录
count_over_time({app_kubernetes_io_instance="admin-service-master-container-web"}[30m]) 

# 12h小时内出现错误的速率
rate({app_kubernetes_io_instance=~".*master-container.*"} |~ "ERROR|error" [12h])
```

#### 集合运算（Built-in aggregation operators）

与PromQL一样，LogQL支持内置聚合运算符的一个子集，可用于聚合单个向量的元素，从而产生具有更少元素但具有集合值的新向量：

- sum：计算标签上的总和
- min：选择最少的标签
- max：选择标签上方的最大值
- avg：计算标签上的平均值
- stddev：计算标签上的总体标准差
- stdvar：计算标签上的总体标准方差
- count：计算向量中元素的数量
- bottomk：通过样本值选择最小的k个元素
- topk：通过样本值选择最大的k个元素

### 模板方法（Template functions）

模板方法目的是用于字符串或数值处理

### IP匹配（Matching IP addresses）

## 告警规则

1.Loki集成了ruler组件，在Loki的配置文件中加上ruler配置：

```yaml
ruler:
    # 触发告警事件后的回调查询地址
    # 如果用grafana的话就配置成grafana/explore
    external_url:
  
    # alertmanager地址 ,如：http://10.0.80.92:9093
    alertmanager_url: <alertmanager_endpoint>
    enable_alertmanager_v2: true
  
    # 启用loki rules API
    enable_api: true
    
    # 对rules分片，支持ruler多实例
    enable_sharding: true
  
    # ruler服务的一致性哈希环配置，用于支持多实例和分片
    ring:
        kvstore:
          # store: inmemory
          store: consul # consul分布式高可用
          consul:
            host: <consul-endpoint>:8500
  
    # rules临时规则文件存储路径
    rule_path: /tmp/rules
   
    # rules规则存储
    # 主要支持本地存储（local）和对象文件系统（azure, gcs, s3, swift）
    storage:
      type: local
      local:
        directory: /loki/rules
      
    # rules规则加载时间
    flush_period: 1m
```

2.告警规则配置
在rules规则目录/loki/rules下新建fake目录，在fake目录下添加告警规则,Loki的规则和prometheus的规则一样：

```yaml
groups:
  # 组名称
  - name: <string>
    rules:
      # Alert名称
      - alert: <string>
        # logQL查询语句
        expr: <string>
        # 产生告警的持续时间
        pending.
        [ for: <duration> | default = 0s ]
        # 自定义告警事件的label
        labels:
        [ <labelname>: <tmpl_string> ]
        # 告警时间的注释
        annotations:
        [ <labelname>: <tmpl_string> ]
```

例子：

```yaml
groups:
- name: nginx_loki
  rules:
  - alert: nginx_5xx_per_30
    expr: sum by(host, remote_addr) (count_over_time({filename="/usr/local/nginx/logs/json_access.log"} | json | status > 200 [5m]))/(sum by(host, remote_addr) (count_over_time({filename="/usr/local/nginx/logs/json_access.log"} | json [5m]))/ 100) > 30
    for: 15s
    labels:
      severity: 1
      team: nginx_loki
    annotations:
      summary: "{{ $labels.host }} 网关， 访问{{ $labels.remote_addr }} 5xx的错误超过30%的情况已超过 15s！"
```

## 调整limit限制
由于我们用聚合函数将日志转成series，对于将日志内容提取到标签中，Loki是有一个默认的长度限制的，我们需要在limits_config中合理调整。合理的调整limits_config将影响我们LogQL v2语句查询结果。

```yaml
limits_config:
  #label的name长度
  max_label_name_length: <int> | default = 1024]
  #label的value长度，这里就是日志内容的最大长度
  max_label_value_length: <int> | default = 2048]
  #label的name个数，如果解释器内容太多则需要调整此配置
  max_label_names_per_series: <int> | default = 30]
  #最大查询series的个数，解释器提取的键值对对应一个唯一的series
  max_query_series: <int> | default = 500]
```

## 参考帖子
- [使用说明的帖子](https://blog.csdn.net/weixin_44275820/article/details/120548004?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-1-120548004.pc_agg_new_rank&utm_term=loki+%E9%85%8D%E7%BD%AE%E8%AF%A6%E7%BB%86%E8%AF%B4%E6%98%8E&spm=1000.2123.3001.4430)
- [grafana loki展示nginx的dashboard](https://blog.csdn.net/d_chunyu/article/details/117203326)