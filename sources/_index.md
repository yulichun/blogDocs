---
title: Overview
weight: 200
---

# 技术

## 概述

文档涉及的监控是应用监控，主要的监控对象是计算机硬件资源、系统、应用。

监控从业务或用途上可以分为指标监控（metrics monitoring）、日志监控（log monitoring）、链路追踪（tracing）。

- 指标监控：将监控项设计为可以量化的指标，根据指标的数值大小确定指标是否存在问题，并且会存储一段时间内的指标值，以便对比指标的趋势。这是最常规的监控方式，zabbix、prometheus、influxdb等监控软件都是这种设计方式。
- 日志监控：日志是监控数据的主要来源，通常日志会作为审计用途，但同时日志也用于监控用途，监控日志的手段通常是通过一些日志数据库分析语法来实现，比如loki、clickhouse
- 链路追踪：链路追踪用于分布式应用系统，可以快速定位问题出现在哪个环节，是什么原因。

监控从使用上通常是图表或者告警通知两种方式。

- 图表:图表是最直观的方式，可以观测指标趋势，了解实时状态，还可以通过饼图、柱状图等查看分布对比情况，常用的图表软件有grafana，我们也使用了完全开源的xobserve
- 告警：告警通知在监控中很重要，相关软件很多，有alertmanager，grafana也有告警通知功能

技术文档会介绍这些监控软件的常规用法和一些基本原理。

- prometheus
  - [搭建简单的监控主机系统](./prometheus/搭建简单的监控主机系统.md)
  - [exporter安装说明](./prometheus/exporter安装说明.md)
  - [一些exporter及其指标说明](./prometheus/一些exporter及其指标说明.md)
  - [prometheus安装说明](./prometheus/prometheus安装说明.md)
  - [prometheus的资源使用量计算指导手册*](./prometheus/prometheus的资源使用量计算指导手册.md)
  - [prometheus高可用方案*](./prometheus/prometheus高可用方案.md)
  - [blackbox-exporter使用说明](./prometheus/blackbox-exporter使用说明.md)
  - [alertManager安装说明](./prometheus/alertManager安装说明.md)
  - [alertManager使用说明](./prometheus/alertManager使用说明.md)
  - [报警处理流程及主要数据结构](./prometheus/报警处理流程及主要数据结构.md)
- grafana
  - [仪表板设计经验总结*](./grafana/仪表板设计经验总结.md)
  - [grafana安装说明](./grafana/grafana安装说明.md)
  - [grafana的告警dashboard定制](./grafana/grafana的告警dashboard定制.md)
  - [grafana定制dashboard](./grafana/grafana定制dashboard.md)
  - [grafana集成到自己的前端](./grafana/grafana集成到自己的前端.md)
  - [grafana插件开发指导](./grafana/grafana插件开发指导.md)
- loki
  - [Loki安装说明](./loki/Loki安装说明.md)
  - [Loki使用说明](./loki/Loki使用说明.md)
  - [官方推荐的loki配置](./loki/官方推荐的loki配置.md)
  - [Loki的记录规则处理日志的疑问*](./loki/Loki的记录规则处理日志的疑问.md)
  - [Loki-针对Nginx日志的实操](./loki/Loki-针对Nginx日志的实操.md)
- TICK
  - [telegraf的一些使用记录](./TICK/telegraf.md)
  - [rsyslog+kafka+telegraf](./TICK/rsyslog+kafka+telegraf.md)
  - [influxdb性能优化](./TICK/influxdb性能优化.md)
  - [influxDb概念详细说明*](./TICK/influxDB概念说明.md)
  - [influxDB在网关上的性能测试](./TICK/influxDB在网关上的性能测试.md)
- 技术对比
  - [ELK和loki的对比](./comparison/ELK和loki的对比.md)
  - [Prometheus与TICK的对比](./comparison/Prometheus与TICK的对比.md)
  - [loki和TICK的性能压测](./comparison/loki和TICK的性能压测.md)
- go语言技术记录
  - [go_template模板文件语法及函数](./go/go_template模板文件语法及函数.md)
- 监控协议
  - [syslog及常用转发中间件的支持](./protocol/syslog及常用转发中间件的支持.md)
- [SRE](./SRE/SRE介绍及在网关上的应用.md)
- xobserve
  - [xobserve目录](./xobserve/_index.md)
  - [xobserve使用](./xobserve/xobserve使用.md)
- clickhouse
  - [clickhouse_clickvisual_安装使用说明](./clickhouse/clickhouse_clickvisual_安装使用说明.md)
- TencentBlueKing
  - [蓝鲸监控操作步骤](./TencentBlueKing/蓝鲸监控操作步骤.md)
  - [蓝鲸调研小结](./TencentBlueKing/蓝鲸调研小结.md)
  - [安装说明](./TencentBlueKing/安装说明.md)
  - [初步调研结论](./TencentBlueKing/初步调研结论.md)
- zabbix
  - [zabbix离线安装说明](./zabbix/zabbix离线安装说明.md)
  - [zabbix操作说明](./zabbix/zabbix操作说明.md)
- rsyslog
  - [rsyslog的安装包源](./rsyslog/rsyslog的安装包源.md)
- issues
  - [grafana问题汇总](./issues/grafana问题汇总.md)
  - [Loki采集问题汇总](./issues/Loki采集问题汇总.md)

# 外部文档的链接

- [nginx-vts-exporter源码](https://github.com/hnlq715/nginx-vts-exporter)
- [prometheus配置详解，中文](https://www.cnblogs.com/zhoujinyi/p/11944176.html)
- [grafana已支持的dashboard](https://grafana.com/grafana/dashboards)
- [alertmanager中邮件、微信、钉钉的详细配置](https://zhuanlan.zhihu.com/p/179294441)
- [一个非常全的告警规则网站](https://awesome-prometheus-alerts.grep.to/)
- 推荐比较好的grafana的dashboard模板：8321、8919、2204、13978、11558、12232
- [一个比较好的介绍prometheus最佳实践的博客](https://www.robustperception.io/blog/)
- [grafana图表制作最佳实践](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/best-practices/)
- [conntrack监控](https://flashcat.cloud/blog/node-exporter-conntrack/)