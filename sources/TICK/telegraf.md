- telegraf官方文档：https://docs.influxdata.com/telegraf/v1.9/
- telegraf的github项目中配置文档：https://github.com/influxdata/telegraf/blob/master/docs/CONFIGURATION.md

# 参数优化
metric_batch_size = 1000 # 采集器每次发生指标个数限制，1000-3000，别太大
flush_interval = “60s” # 刷新数据写入到output时间间隔，在压力大的情况下，flush_interval大一点数据入库更快，但是flush_interval太大可能造成提交不及时，所以还是根据场景设定

