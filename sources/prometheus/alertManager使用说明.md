## 概述

Alertmanager 主要用于接收Prometheus 发送的告警信息，它支持丰富的告警通知渠道，而且很容易做到告警信息进行分组、静默、抑制等，是Prometheus生态中非常重要的一个核心模块。
Prometheus 告警分为两部分：

- Prometheus服务器中的警报规则向Alertmanager发送告警。
- Alertmanager 管理这些告警，对这些告警进行分组去重，根据理由规则发送到接受者，比如发送电子邮件、呼叫通知系统，以及即时通讯平台。

## 核心概念

### 分组

分组将类似性质的警报分类为单个通知。当许多系统同时发生故障并且可能同时触发数百到数千个警报时，这在较大的中断期间尤其有用。

示例：当发生网络分区故障时，集群中正在运行数十个或数百个服务实例。您的一半服务实例无法再访问数据库。Prometheus 中的警报规则被配置为在每个服务实例无法与数据库通信时发送警报。因此，数百个警报被发送到 Alertmanager。

作为用户，您只想获得一个页面，同时仍然能够准确查看哪些服务实例受到影响。因此，可以将 Alertmanager 配置为按集群和警报名称对警报进行分组，以便它发送单个紧凑通知。

警报分组、分组通知的时间以及这些通知的接收者由配置文件中的路由树进行配置。

### 抑制

抑制是一个概念，如果某些其他警报已经触发，则抑制某些警报的通知。

示例：发出警报，通知无法访问整个集群。如果该特定警报正在触发，Alertmanager 可以配置为静音与该集群相关的所有其他警报。这可以防止收到与实际问题无关的数百或数千个触发警报的通知。

通过 Alertmanager 的配置文件进行配置。

### 静默

静默是在给定时间内简单地将警报静音的直接方法。静默是基于匹配器配置的，就像路由树一样。检查传入警报是否与活动静默的所有相等或正则表达式匹配器匹配。如果他们这样做，则不会发送该警报的通知。

静默是在 Alertmanager 的 Web 界面中配置的。

### 客户行为

Alertmanager对其客户端的行为有特殊要求。这些仅与不使用 Prometheus 发送警报的高级用例相关。

### 高可用性

Alertmanager 支持配置以创建集群以实现高可用性。这可以使用–cluster-*标志进行配置。

重要的是不要在 Prometheus 及其警报管理器之间负载平衡流量，而是将 Prometheus 指向所有警报管理器的列表。

## 支持的告警方式

- email
- pagerduty
- pushover
- slack
- opsgenie
- webhook
- victorops
- wechat（微信）

## Alertmanager配置项说明

主要分4块配置：
- global: 全局配置，包括报警解决后的超时时间、SMTP 相关配置、各种渠道通知的 API 地址等等。

- route: 用来设置报警的分发策略，它是一个树状结构，按照深度优先从左向右的顺序进行匹配。

- receivers: 配置告警消息接受者信息，例如常用的 email、wechat、slack、webhook 等消息通知方式。

- inhibit_rules: 抑制规则配置，当存在与另一组匹配的警报（源）时，抑制规则将禁用与一组匹配的警报（目标）。

详细配置说明：

```yaml
# 配置告警通知发送者
global:
  resolve_timeout: 5m # 在此时间过后，如果警报尚未更新，则可以将其声明为已解决,并发送已解决通知给接收者
  ## ------这一段是邮箱方式需要设置的-----start
  ## 源邮箱
  smtp_from: '85345569@qq.com'
  ## 源邮箱要设置smtp
  smtp_smarthost: 'smtp.qq.com:465'
  ## 源邮箱要设置smtp账号
  smtp_auth_username: '853455693@qq.com'
  ## 源邮箱要设置smtp密码
  smtp_auth_password: 'yugiqnklgchxbddh'
  ## 源邮箱要设置smtp是否使用tls
  smtp_require_tls: false
  ## 源邮箱要设置smtp的hello信息
  smtp_hello: 'qq.com'
  ## ------这一段是邮箱方式需要设置的-----end

# 告警通知路由，配置组
route:
group_by: ['alertname']  # 报警分组
group_wait: 5s  # 当收到告警的时候，等待5秒确认时间内是否有新告警，如有则一并发送
group_interval: 5s # Alert Manager 会每隔 5s 巡视一下 Group，看看是否需要发送 notification
repeat_interval: 5m # 发送报警间隔，如果指定时间内没有修复，则重新发送报警。通常设置较大
receiver: developer # 默认接收者
routes: # 路由，根据标签匹配情况将告警路由到不同的接收者
  - match:
      instanceType: 'GW'
    receiver: services
  - match:
      instanceType: 'node'
    receiver: developer

# 告警通知接收者
receivers:
- name: 'developer'
  # 邮件配置
  email_configs:
  - to: '9935226@qq.com'
    send_resolved: true
  # 钉钉配置
  webhook_configs:
  # 发送到prometheus-webhook-dingtalk
  - url: http://prometheus-webhook-dingtalk:8060/dingtalk/developer/send
  # 企业微信配置
  wechat_configs:
    # 企业信息("我的企业"--->"CorpID"[在底部])
  - corp_id: 'wwxxxxxxxxxxxxxx'
    # 企业微信url
    api_url: 'https://qyapi.weixin.qq.com/cgi-bin/'
    send_resolved: true
    to_party: '2'
    # 企业微信("企业应用"-->"自定应用"[Prometheus]--> "AgentId")
    agent_id: '1000002'
    # 企业微信("企业应用"-->"自定应用"[Prometheus]--> "Secret")
    api_secret: '1FvHxuGbbG35FYsuW0YyI4czWY/.2'
    
- name: 'services'
  # 钉钉配置，发送到prometheus-webhook-dingtalk
  webhook_configs:
  - url: http://prometheus-webhook-dingtalk:8060/dingtalk/services/send


# 抑制器配置
inhibit_rules:
# 相同alertname，取级别高的进行告警
- source_match:
    severity: '1'
  target_match:
    severity: '2'
  equal: ['alertname']
  # 网关节点后端服务的告警通知抑制网关节点代理服务的告警通知
- source_match_re:
    alertname: GWNode[Http|Tcp]ProxyUpstream.*
  target_match_re:
    alertname: GWNode[Http|Tcp]ProxyServer.*
  equal:
  - instance
  - server_description
  - zone
```

- [官方配置说明](https://prometheus.io/docs/alerting/latest/configuration/)
- [配置说明相关博客](https://blog.csdn.net/qq_43437874/article/details/120386208)
