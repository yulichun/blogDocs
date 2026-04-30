1. 拉取alertmanager的docker镜像

```shell
$ docker pull prom/alertmanager:latest
```
2. 创建/opt/pro_config/alertmanager/alertmanager.yml，用于映射到alertmanager容器的/etc/alertmanager/alertmanager.yml，并配置以下内容：

```yaml
global:
  resolve_timeout: 5m
  smtp_from: '{qq号}@qq.com'
  smtp_smarthost: 'smtp.qq.com:465'
  smtp_auth_username: '{qq号}@qq.com'
  smtp_auth_password: '{密码}'
  smtp_require_tls: false
  smtp_hello: 'qq.com'
route:
  group_by: ['alertname']
  group_wait: 5s
  group_interval: 5s
  repeat_interval: 5m
  receiver: 'email'
receivers:
  - name: 'email'
    email_configs:
      - to: '{邮箱地址}'
        send_resolved: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

4. 创建prometheus容器

```shell
$ docker run --name alertmanager -d -p 9093:9093 -v /opt/pro_config/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml prom/alertmanager
```

5. 访问“http://{IP}:9093”