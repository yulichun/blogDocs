# 告警定制的三种方式

- 利用prometheus采集alertmanager的metrics，grafana以prometheus作为数据源
- grafana集成alertmanager数据源
- 使用grafana的mysql数据源

三种方式分别以prometheus、alertmanager、mysql作为数据源来制作dashboard。

# 利用prometheus采集alertmanager的metrics，grafana以prometheus作为数据源

这个方式是最通用的方式，因为：首先，一般告警规则是再prometheus侧定义；其次，grafana对prometheus有良好的支持。

步骤：

1. 安装启动alertmanager，参考[alertmanager安装说明](alertManager安装说明.md)和[alertmanager使用说明](alertManager使用说明.md)
2. 在prometheus配置采集alertmanager的job：
```yaml
- job_name: 'alertmanager'
    static_configs:
      - targets: ['alertmanager地址:9093']
```
3. 在grafana添加prometheus数据源
4. grafana中导入dashboard，dashboard地址：https://grafana.com/grafana/dashboards/9578

# grafana集成alertmanager数据源

1. 下载alertmanager插件

```shell
$ cd /usr/local/grafana/bin
$ ./grafana-cli plugins install camptocamp-prometheus-alertmanager-datasource
```

2. 修改grafana配置文件

```shell
$ cd /usr/local/grafana/conf/
$ vim defaults.ini  #将plugins地址修改为/var/lib/grafana/plugins
$ plugins = /var/lib/grafana/plugins
```

3. 重启

```shell
$ cd /usr/local/grafana
$ nohup ./bin/grafana-server &
```

4. 进入grafana界面添加alertmanager数据源
5. 在grafana界面导入dashboard，dashboard地址：https://grafana.com/grafana/dashboards/8010

# 使用grafana的mysql数据源

该方式需要grafana使用mysql，然后grafana集成该mysql数据源，利用其中的数据展示告警dashboard。

1. 配置grafana的配置文件default.ini
```properties
GF_DATABASE_TYPE: "mysql"
GF_DATABASE_URL: "mysql://USER:PASS@HOST:3306/DB"
```
2. 在grafana的界面配置grafana的mysql为数据源
   
   Type: Mysql
   Host: HOST
   Database: DB
   User: USER
   Password: PASS
   
3. 导入dashboard，dashboard地址为：https://grafana.com/grafana/dashboards/3489

