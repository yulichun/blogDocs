# blogDocs

网关与监控相关技术笔记：指标、日志、告警与可视化工具的选型、部署与使用记录。

- **在线阅读（GitHub Pages）**：<https://yulichun.github.io/blogDocs/>
- **概述与导航**： [sources/_index.md](sources/_index.md)

---

## 文档索引

### [技术概述](sources/_index.md)

#### prometheus

- [搭建简单的监控主机系统](sources/prometheus/搭建简单的监控主机系统.md)
- [exporter安装说明](sources/prometheus/exporter安装说明.md)
- [一些exporter及其指标说明](sources/prometheus/一些exporter及其指标说明.md)
- [prometheus安装说明](sources/prometheus/prometheus安装说明.md)
- [prometheus的资源使用量计算指导手册](sources/prometheus/prometheus的资源使用量计算指导手册.md)
- [prometheus高可用方案](sources/prometheus/prometheus高可用方案.md)
- [blackbox-exporter使用说明](sources/prometheus/blackbox-exporter使用说明.md)
- [alertManager安装说明](sources/prometheus/alertManager安装说明.md)
- [alertManager使用说明](sources/prometheus/alertManager使用说明.md)
- [报警处理流程及主要数据结构](sources/prometheus/报警处理流程及主要数据结构.md)

#### grafana

- [仪表板设计经验总结](sources/grafana/仪表板设计经验总结.md)
- [grafana安装说明](sources/grafana/grafana安装说明.md)
- [grafana的告警dashboard定制](sources/grafana/grafana的告警dashboard定制.md)
- [grafana定制dashboard](sources/grafana/grafana定制dashboard.md)
- [grafana集成到自己的前端](sources/grafana/grafana集成到自己的前端.md)
- [grafana插件开发指导](sources/grafana/grafana插件开发指导.md)

#### loki

- [Loki安装说明](sources/loki/Loki安装说明.md)
- [Loki使用说明](sources/loki/Loki使用说明.md)
- [官方推荐的loki配置](sources/loki/官方推荐的loki配置.md)
- [Loki的记录规则处理日志的疑问](sources/loki/Loki的记录规则处理日志的疑问.md)
- [Loki-针对Nginx日志的实操](sources/loki/Loki-针对Nginx日志的实操.md)

#### TICK

- [telegraf的一些使用记录](sources/TICK/telegraf.md)
- [rsyslog+kafka+telegraf](sources/TICK/rsyslog+kafka+telegraf.md)
- [influxdb性能优化](sources/TICK/influxdb性能优化.md)
- [influxDB概念详细说明](sources/TICK/influxDB概念说明.md)
- [influxDB在网关上的性能测试](sources/TICK/influxDB在网关上的性能测试.md)

#### 技术对比

- [ELK和loki的对比](sources/comparison/ELK和loki的对比.md)
- [Prometheus与TICK的对比](sources/comparison/Prometheus与TICK的对比.md)
- [loki和TICK的性能压测](sources/comparison/loki和TICK的性能压测.md)

#### go语言技术记录

- [go_template模板文件语法及函数](sources/go/go_template模板文件语法及函数.md)

#### 监控协议

- [syslog及常用转发中间件的支持](sources/protocol/syslog及常用转发中间件的支持.md)

#### 其他

- [SRE介绍及在网关上的应用](sources/SRE/SRE介绍及在网关上的应用.md)

#### xobserve

- [xobserve](sources/xobserve/_index.md)
- [xobserve使用](sources/xobserve/xobserve使用.md)

#### clickhouse

- [clickhouse_clickvisual_安装使用说明](sources/clickhouse/clickhouse_clickvisual_安装使用说明.md)

#### TencentBlueKing

- [蓝鲸监控操作步骤](sources/TencentBlueKing/蓝鲸监控操作步骤.md)
- [蓝鲸调研小结](sources/TencentBlueKing/蓝鲸调研小结.md)
- [安装说明](sources/TencentBlueKing/安装说明.md)
- [初步调研结论](sources/TencentBlueKing/初步调研结论.md)

#### zabbix

- [zabbix离线安装说明](sources/zabbix/zabbix离线安装说明.md)
- [zabbix操作说明](sources/zabbix/zabbix操作说明.md)

#### rsyslog

- [rsyslog的安装包源](sources/rsyslog/rsyslog的安装包源.md)

#### issues

- [问题汇总](sources/issues/_index.md)
- [grafana问题汇总](sources/issues/grafana问题汇总.md)
- [Loki采集问题汇总](sources/issues/Loki采集问题汇总.md)

---

## 外部参考链接

- [nginx-vts-exporter源码](https://github.com/hnlq715/nginx-vts-exporter)
- [prometheus配置详解（中文）](https://www.cnblogs.com/zhoujinyi/p/11944176.html)
- [grafana已支持的dashboard](https://grafana.com/grafana/dashboards)
- [alertmanager中邮件、微信、钉钉的详细配置](https://zhuanlan.zhihu.com/p/179294441)
- [告警规则汇总（awesome-prometheus-alerts）](https://awesome-prometheus-alerts.grep.to/)
- [Robust Perception（Prometheus 实践）](https://www.robustperception.io/blog/)
- [grafana图表制作最佳实践](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/best-practices/)
- [conntrack监控](https://flashcat.cloud/blog/node-exporter-conntrack/)

推荐 Grafana Dashboard 模板 ID：8321、8919、2204、13978、11558、12232。

---

## 本地构建 GitBook（Honkit）

```bash
npm ci
npm run build
```

生成站点目录：`_book`。预览：`npm run serve`。
