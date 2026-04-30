# 概述

Syslog常被称为系统日志或系统记录，是一种用来在互联网协议（TCP/IP）的网上中传递记录档消息的标准。这个词汇常用来指涉实际的syslog 协议，或者那些提交syslog消息的应用程序或数据库。

# 基于网络协议

- UDP
- TCP

# 数据结构

syslog是纯明文报文，数据结构如下，这是根据rfc-5414文档翻译而来：

```
# 一条信息的构成
SYSLOG-MSG = HEADER SP STRUCTURED-DATA [SP MSG]  # 最后的MSG是可省略的
# HEADER = 优先级 版本 空格 时间戳 空格 主机名 空格 应用名 空格 进程id 空格 信息id
HEADER = PRI VERSION SP TIMESTAMP SP HOSTNAME SP APP-NAME SP PROCID SP MSGID
# PRI优先级
PRI = "<" PRIVAL ">" # 优先级 <0>
# PRI优先级的值
PRIVAL = 1*3DIGIT ; range 0 .. 191 # 3位数字, 0到191
# syslog版本号
VERSION = NONZERO-DIGIT 0*2DIGIT # 默认为 RFC5424默认为1
# 主机名
HOSTNAME = NILVALUE / 1*255PRINTUSASCII # - 或 255位可打印ASCII值
# 应用名
APP-NAME = NILVALUE / 1*48PRINTUSASCII # - 或 48位可打印ASCII值
# 进程ID
PROCID = NILVALUE / 1*128PRINTUSASCII # - 或 128位可打印ASCII值
# 信息ID
MSGID = NILVALUE / 1*32PRINTUSASCII # - 或 32位可打印ASCII值
# 时间戳
TIMESTAMP = NILVALUE / FULL-DATE "T" FULL-TIME # - 或 "0000-00-00"
# 完整日期格式
FULL-DATE = DATE-FULLYEAR "-" DATE-MONTH "-" DATE-MDAY # "0000-00-00"
# 年
DATE-FULLYEAR = 4DIGIT # 四位数字
# 月
DATE-MONTH = 2DIGIT ; 01-12 # 两位数字
# 日
DATE-MDAY = 2DIGIT ; 01-28, 01-29, 01-30, 01-31 based on month/year
# 完整时间（带时区）
FULL-TIME = PARTIAL-TIME TIME-OFFSET
# 时间（不带时区）
PARTIAL-TIME = TIME-HOUR ":" TIME-MINUTE ":" TIME-SECOND # 23:59:59
[TIME-SECFRAC]
# 小时
TIME-HOUR = 2DIGIT ; 00-23 # 两位数字
# 分
TIME-MINUTE = 2DIGIT ; 00-59 # 两位数字
# 秒
TIME-SECOND = 2DIGIT ; 00-59 # 两位数字
# 时间的小数部分
TIME-SECFRAC = "." 1*6DIGIT # 6位数字
TIME-OFFSET = "Z" / TIME-NUMOFFSET # 相对于标准时区的偏移， "Z" 或 +/- 23:59
# 相对于便准时区的偏移
TIME-NUMOFFSET = ("+" / "-") TIME-HOUR ":" TIME-MINUTE # +/- 23:59
# 结构化数据
STRUCTURED-DATA = NILVALUE / 1*SD-ELEMENT # - 或 SD-ELEMENT
SD-ELEMENT = "[" SD-ID *(SP SD-PARAM) "]" # [SD-ID*( PARAM-NAME="PARAM-VALUE")]
SD-PARAM = PARAM-NAME "=" %d34 PARAM-VALUE %d34 # PARAM-NAME="PARAM-VALUE"
SD-ID = SD-NAME # SD-ID
PARAM-NAME = SD-NAME # 参数名
PARAM-VALUE = UTF-8-STRING # utf-8字符， '"', '\' 和 ']'必须被转义
SD-NAME = 1*32PRINTUSASCII # 1到32位可打印ascii值，除了'=',空格, ']', 双引号(")
MSG = MSG-ANY / MSG-UTF8 # 信息
MSG-ANY = *OCTET ; not starting with BOM # 八进制字符串 不以BOM开头
MSG-UTF8 = BOM UTF-8-STRING # utf-8格式字符串
BOM = %xEF.BB.BF # 表明编码方式，以 EF BB BF开头表明utf-8编码
UTF-8-STRING = *OCTET # RFC 3629规定的字符
OCTET = %d00-255 # ascii
SP = %d32 # 空格
PRINTUSASCII = %d33-126 # ascii值的33-126，即数字、大小写字母、标点符号
NONZERO-DIGIT = %d49-57 # ascii的49-57
DIGIT = %d48 / NONZERO-DIGIT # ascii的48-57
NILVALUE = "-" # 无对应值
```
![syslog报文](./images/syslog%E6%8A%A5%E6%96%87.png)

- [参考文档](https://blog.csdn.net/chenwr2018/article/details/121742978)

# 中间件对syslog的支持

## rsyslog

### 接入数据
配置文件中配置接入syslog
```
# 接入syslog，514端口可以接收udp和tcp的syslog数据
module(load="imudp")
input(type="imudp" address="127.0.0.1" port="514")

module(load="imtcp")
input(type="imtcp" address="127.0.0.1" port="514")

```

### 解析数据，配置转发数据格式
rsyslog接收到数据后，会将数据根据syslog规则进行解析，获得如下字段：
PRI
VERSION
TIMESTAMP
HOSTNAME
APP-NAME
PROCID
MSGID
STRUCTURED-DATA
msg
...
这些字段的命名同rfc5424协议的字段命名一致。

rsyslog中利用这些字段可以组织转发内容，利用的是template功能，比如下面这个template，将转发内容又格式化为syslog格式了：
```
template(name="RSYSLOG_SyslogProtocol23Format" type="string"
     string="<%PRI%>1 %TIMESTAMP:::date-rfc3339% %HOSTNAME% %APP-NAME% %PROCID% %MSGID% %STRUCTURED-DATA% %msg%\n")
```

### 转发数据
转发syslog数据到其他syslog服务，template（数据格式）使用上面的RSYSLOG_SyslogProtocol23Format：
```
action(type="omfwd" Protocol="udp" TCP_Framing="octet-counted" Target="127.0.0.1" Port="1514" Template="RSYSLOG_SyslogProtocol23Format")
```

## telegraf

### 接入数据
```
[[inputs.syslog]]
server = "udp://127.0.0.1:1514"
```

### 处理syslog数据
telegraf在接收syslog数据后会自动解析syslog数据结构：
```
{
    measurement: 'syslog'
    tag: {
        pri: 1
        severity: 120
        appname： 'user_access'
        ... // 跟rfc5424协议定义的字段一致
    }
}
```
如何去处理syslog数据通常是根据需求定的，这里我们根据nginx的场景处理，场景是这样的：有user_access、user_action、user_admin、alert这4种日志，日志的tag分别是它们的名称，需要将它们分别存储到influxDB的4张表中。
处理配置如下：
```
[[processors.rename]]
  # 筛选syslog日志
  namepass = ["syslog"]
  ## Specify one sub-table per rename operation.
  [[processors.rename.replace]]
    measurement = "syslog"
    # 将筛选出来的日志的measurement修改为user_access
    dest = "user_access"
    # 筛选appname是user_access的日志
    [processors.rename.tagpass]
      appname = ["user_access"]
  # 解析message字段
  [[processors.parser]]
    parse_fields = ["message"]
    merge = "override"
    data_format = "logfmt"

[[processors.rename]]
  [[processors.rename.replace]]
    measurement = "syslog"
    dest = "admin_action"
    [processors.rename.tagpass]
      appname = ["admin_action"]
  [[processors.parser]]
    parse_fields = ["message"]
    merge = "override"
    data_format = "logfmt"

[[processors.rename]]
  [[processors.rename.replace]]
    measurement = "syslog"
    dest = "user_action"
    [processors.rename.tagpass]
      appname = ["user_action"]
  [[processors.parser]]
    parse_fields = ["message"]
    merge = "override"
    data_format = "logfmt"

[[processors.rename]]
  [[processors.rename.replace]]
    measurement = "syslog"
    dest = "alert"
    [processors.rename.tagpass]
      appname = ["alert"]
  [[processors.parser]]
    parse_fields = ["message"]
    merge = "override"
    data_format = "logfmt"
```

### 输出处理后数据
接着上面的场景，输出处理后的数据到influxDB：
```
[[outputs.influxdb]]
urls = ["http://127.0.0.1:64300"]
database = "telegraf"
# 指定表名
retention_policy = "user_access"
# 去除这些字段
fielddrop = ["message", "timeQuality*", "version", "severity_code", "facility_code", "procid", "msgid", "sdid"]
# measurement为user_access的数据输出到influxDB
namepass = ["user_access"]
# 去除这些tag
tagexclude = ["message", "severity", "facility", "hostname", "appname"]
```

## promtail

```
scrape_configs:
- job_name: syslog
  # syslog服务，监听1514端口
  syslog:
    listen_address: 127.0.0.1:1514
    labels:
      job: "syslog"
  # 接收到syslog数据会根据syslog的字段自动解析为“__syslog_message_字段名称”字样的label，我们这里将__syslog_message_hostname和__syslog_message_app_name转变为host和appname
  relabel_configs:
    - source_labels: ['__syslog_message_hostname']
      target_label: 'host'
    - source_labels: ['__syslog_message_app_name']
      target_label: 'appname'
```


## rsyslog、telegraf和promtail的比较

- 相同点：
  - 都是转发和采集用途的中间件，带有一定的解析能力
  - 都会将接收到的syslog数据按照syslog协议中的字段进行解析
- 不同点：
  - telegraf可以采集指标
  - 数据处理上，telegraf功能更丰富，并且可以聚合计算
  - 速度上rsyslog快并且资源占用小
  - promtail是专门为loki日志数据库实现的