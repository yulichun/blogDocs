# 什么是 blackbox exporter?

Blackbox Exporter 是 Prometheus 社区提供的 官方黑盒监控解决方案,其允许用户通过: http\HTTPS\DNS\TCP\ICMP的方式对网络进行探测.

# docker方式安装blackbox exporter

- 拉取镜像
```shell
docker pull prom/blackbox-exporter
```

- 运行 blackbox exporter

```shell
docker run -id --name blackbox-exporter -p 9115:9115  prom/blackbox-exporter
```

# blackbox exporter 配置文件解读
- [官方详细配置文件解释](https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md)
- [官方默认配置文件](https://github.com/prometheus/blackbox_exporter/blob/master/blackbox.yml)
- [官方默认配置文件](https://github.com/prometheus/blackbox_exporter/blob/master/example.yml)

建议看官方详细配置文件解释，比较全面。

运行 blackbox exporter 时,需要用户提供探针的配置信息,这些配置信息可能是一些自定义的 HTTP 头信息,也可能是探测时需要的一些 TSL(秘钥证书) 配置,也可能是探针本身的验证行为.在 blackbox exporter 每一个探针配置称为一个 module,并且以 YAML 配置文件的形式提供给 blackbox exporter.每一个 module 主要包含以下配置内容,探针类型(prober),验证访问超时时间(timeout),以及当前探针的具体配置项:

```yaml
#探针类型: http https tcp dns icmp
prober: <prober_string>   #必选

#超时时间:
[timeout: <duration>] #默认单位秒

#探针的详细配置,最多只能配置其中一个
[ http: <http_probe> ]
[ tcp: <tcp_probe> ]
[ dns: <dns_probe> ]
[ icmp: <icmp_probe> ]

<http_probe>可配置参数
# 此探针接受的状态代码。 默认为2xx。
[ valid_status_codes: <int>, ... | default = 2xx ]

# 此探针接受的 HTTP 版本.
[ valid_http_versions: <string>, ... ]

#探针将使用的HTTP方法。
[ method: <string> | default = "GET" ]

# 为探针设置的HTTP标头。
headers:
[ <string>: <string> ... ]

# 探针是否将遵循任何重定向
[ no_follow_redirects: <boolean> | default = false ]

# 如果存在SSL，则探测失败。
[ fail_if_ssl: <boolean> | default = false ]

# 如果不存在SSL，则探测失败。
[ fail_if_not_ssl: <boolean> | default = false ]

# 如果响应主体与正则表达式匹配，则探测失败。
fail_if_body_matches_regexp:
[ - <regex>, ... ]

# 如果响应主体与正则表达式不匹配，则探测失败。
fail_if_body_not_matches_regexp:
[ - <regex>, ... ]

# 如果响应头与正则表达式匹配，则探测失败。 对于具有多个值的标头，如果*至少一个*匹配，则失败。
fail_if_header_matches:
[ - <http_header_match_spec>, ... ]

# 如果响应头与正则表达式不匹配，则探测失败。 对于具有多个值的标头，如果* none *不匹配，则失败。
fail_if_header_not_matches:
[ - <http_header_match_spec>, ... ]

# HTTP探针的TLS协议的配置。
tls_config:
[ <tls_config> ]

# 目标的HTTP基本身份验证凭据。
basic_auth:
[ username: <string> ]
[ password: <secret> ]

# 目标的承载令牌。
[ bearer_token: <secret> ]

# 目标的承载令牌文件
[ bearer_token_file: <filename> ]

# 用于连接到目标的HTTP代理服务器。
[ proxy_url: <string> ]

# HTTP探针的IP协议（ip4，ip6）
[ preferred_ip_protocol: <string> | default = "ip6" ]
[ ip_protocol_fallback: <boolean> | default = true ]

# 探针中使用的HTTP请求的主体。
body: [ <string> ]


###################################################################
<http_header_match_spec>
header: <string>,
regexp: <regex>,
[ allow_missing: <boolean> | default = false ]
```

# 几种应用场景

## ping 检测

- 可以通过 ping(icmp)检测服务器的存活,在 prometheus 配置文件中配置使用 ping module:

```yaml
icmp:
prober: icmp
```

- 与 prometheus 集成

```yaml
- job_name: 'blackbox-ping'
  metrics_path: /probe
  params:
  modelus: [icmp]
  static_configs:
    - targets:
        - 223.5.5.5
          lables:
          instance: aliyun
    - targets:
        - 47.92.229.67
          lables:
          instance: zsf
          relabel_configs:
        - source_labels: [__address__]
          target_label: __param_target
        - source_labels: [__param_target]
          target_label: instance
        - target_label: __address__
          replacement: 192.168.111.65:9115
```

## HTTP

- blackbox config file

```yaml
modules:
http_2xx:
prober: http
http:
method: GET
http_post_2xx:
prober: http
http:
method: POST
```

- 与 prometheus 集成,采用prometheus 的 Relabelinng 能力(服务发现)

```yaml
- job_name: 'blackbox-http'
  metrics_path: /probe
  params:
  modelue: [http_2xx]
  static_configs:
    - targets:
        - http://www.zhangshoufu.com
        - http://www.xuliangwei.com
          relabel_configs:
        - source_labels: [__address__]
          target_label: __param_target
        - source_labels: [__param_target]
          target_label: instance
        - target_label: __address__
          replacement: 192.168.111.65:9115  #blackbox-exporter 所在的机器和端口
```

这里针对每一个探针服务（如http_2xx）定义一个采集任务，并且直接将任务的采集目标定义为我们需要探测的站点。在采集样本数据之前通过relabel_configs对采集任务进行动态设置。

1. 根据 Target 实例的地址,写入__param_target 标签中,__param_<name>形式的标签表示,在采集任务时会在请求目标地址中添加<name>参数,等同于 params 的设置
2. 获取__param_target的值，并覆写到instance标签中；
3. 覆写Target实例的__address__标签值为BlockBox Exporter实例的访问地址。

## 自定义 HTTP 请求

1. HTTP服务通常会以不同的形式对外展现，有些可能就是一些简单的网页，而有些则可能是一些基于REST的API服务。 对于不同类型的HTTP的探测需要管理员能够对HTTP探针的行为进行更多的自定义设置，包括：HTTP请求方法、HTTP头信息、请求参数等。对于某些启用了安全认证的服务还需要能够对HTTP探测设置相应的Auth支持。对于HTTPS类型的服务还需要能够对证书进行自定义设置。
如下所示，这里通过method定义了探测时使用的请求方法，对于一些需要请求参数的服务，还可以通过headers定义相关的请求头信息，使用body定义请求内容：

```yaml
http_post_2xx:
prober: http
timeout: 5s
http:
method: POST
headers:
Content-Type: application/json
body: '{}'
```

2. 如果HTTP服务启用了安全认证，Blockbox Exporter内置了对basic_auth的支持，可以直接设置相关的认证信息即可：

```yaml
http_basic_auth_example:
prober: http
timeout: 5s
http:
method: POST
headers:
Host: "login.example.com"
basic_auth:
username: "username"
password: "mysecret"
```

1. 对于使用了Bear Token的服务也可以通过bearer_token配置项直接指定令牌字符串，或者通过bearer_token_file指定令牌文件。
对于一些启用了HTTPS的服务，但是需要自定义证书的服务，可以通过tls_config指定相关的证书信息：

```yaml
http_custom_ca_example:
prober: http
http:
method: GET
tls_config:
ca_file: "/certs/my_cert.crt"
```

## 自定义探针行为

1. 在默认情况下HTTP探针只会对HTTP返回状态码进行校验，如果状态码为2XX（200 <= StatusCode < 300）则表示探测成功，并且探针返回的指标probe_success值为1。
如果用户需要指定HTTP返回状态码，或者对HTTP版本有特殊要求，如下所示，可以使用valid_http_versions和valid_status_codes进行定义：

```yaml
http_2xx_example:
prober: http
timeout: 5s
http:
valid_http_versions: ["HTTP/1.1", "HTTP/2"]
valid_status_codes: [200,301,302]
```

2. 默认情况下，Blockbox返回的样本数据中也会包含指标probe_http_ssl，用于表明当前探针是否使用了SSL：

```yaml
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 0
```

3. 而如果用户对于HTTP服务是否启用SSL有强制的标准。则可以使用fail_if_ssl和fail_if_not_ssl进行配置。fail_if_ssl为true时，表示如果站点启用了SSL则探针失败，反之成功。fail_if_not_ssl刚好相反。

```yaml
http_2xx_example:
prober: http
timeout: 5s
http:
valid_status_codes: []
method: GET
no_follow_redirects: false
fail_if_ssl: false
fail_if_not_ssl: false
```

4. 除了基于HTTP状态码，HTTP协议版本以及是否启用SSL作为控制探针探测行为成功与否的标准以外，还可以匹配HTTP服务的响应内容。使用fail_if_matches_regexp和fail_if_not_matches_regexp用户可以定义一组正则表达式，用于验证HTTP返回内容是否符合或者不符合正则表达式的内容。

```yaml
http_2xx_example:
prober: http
timeout: 5s
http:
method: GET
fail_if_matches_regexp:
- "Could not connect to database"
  fail_if_not_matches_regexp:
    - "Download the latest version here"
```

# url访问metrics以及metrics介绍

访问http://localhost:9115/probe?target=google.com&module=http_2xx可以返回http_2xx模块针对目标google.com的探测结果，
结果大致如下：

```properties
#DNS解析时间,单位 s
probe_dns_lookup_time_seconds 0.039431355
#探测从开始到结束的时间,单位 s,请求这个页面响应时间
probe_duration_seconds 0.651619323

probe_failed_due_to_regex 0

#HTTP 内容响应的长度
probe_http_content_length -1
#按照阶段统计每阶段的时间
probe_http_duration_seconds{phase="connect"} 0.050388884   #连接时间
probe_http_duration_seconds{phase="processing"} 0.45868667 #处理请求的时间
probe_http_duration_seconds{phase="resolve"} 0.040037612  #响应时间
probe_http_duration_seconds{phase="tls"} 0.145433254    #校验证书的时间
probe_http_duration_seconds{phase="transfer"} 0.000566269
#重定向的次数
probe_http_redirects 1
#ssl 指示是否将 SSL 用于最终重定向
probe_http_ssl 1
#返回的状态码
probe_http_status_code 200
#未压缩的响应主体长度
probe_http_uncompressed_body_length 40339
#http 协议的版本
probe_http_version 1.1
#使用的 ip 协议的版本号
probe_ip_protocol 4

probe_ssl_earliest_cert_expiry 1.59732e+09
#是否探测成功
probe_success 1
#TLS 的版本号
probe_tls_version_info{version="TLS 1.2"} 1
```