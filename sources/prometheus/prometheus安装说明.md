1. 拉取prometheus的docker镜像

```shell
$ docker pull prom/prometheus:latest
```
2. 创建/opt/pro_config/prometheus/prometheus.yml，用于映射到prometheus容器的/etc/prometheus/prometheus.yml，并配置以下内容：

```yaml
global:
  scrape_interval:     60s
  evaluation_interval: 60s
 
# exporter配置
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: prometheus
 
  - job_name: nginx
    static_configs:
      - targets: ['10.0.249.177:9913']
        labels:
          instance: nginx

  - job_name: node
    static_configs:
      - targets: ['10.0.249.177:9100']
        labels:
          instance: node

# 配置alertManager
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 192.168.196.128:9093

# 配置监控规则文件
rule_files:
  - "/usr/local/prometheus/rules/*.rules"
```

3. 创建监控规则文件夹创建/opt/pro_config/prometheus/rules文件夹，用于映射到prometheus容器的/usr/local/prometheus/rules/ ，文件夹下创建nginx.rules，并配置以下内容：

```yaml
groups:
- name: nginx
  rules:
  - alert: nginx_server_connections_large
    expr: nginx_server_connections{instance="nginx", job="nginx", status="accepted"} >= 10
    for: 15s
    labels:
      severity: 1
      team: nginx
    annotations:
      summary: "{{ $labels.instance }} nginx的后台连接超过10个的情况已超过 15s！"
```

4. 创建prometheus容器

```shell
$docker run  -d -p 9090:9090 -v /opt/pro_config/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -v /opt/pro_config/prometheus/rules/:/usr/local/prometheus/rules/ prom/prometheus
```

5. 访问“http://{IP}:9090”