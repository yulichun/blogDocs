# node-exporter

## 安装包方式

### 下载

- [下载页](https://prometheus.io/download/)

### 安装

```shell
$ mkdir /opt/k8s/prometheus & cd /opt/k8s/prometheus
$ wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
$ tar -xzvf node_exporter-0.18.1.linux-amd64.tar.gz
$ cd node_exporter-0.18.1.linux-amd64

```

### 启动

```shell
$ cd /opt/k8s/prometheus/node_exporter-0.18.1.linux-amd64
$ ./node_exporter
```

### 访问地址

```
http://10.0.249.177:9100/metrics
```

## docker-compose方式

### docker-compose配置

创建docker-compose.yml，配置文件内容：
```yaml
version: '3'

services:
  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    restart: always
    deploy:
      resources:
         limits:
            cpus: "2.00"
            memory: 500M
         reservations:
            memory: 200M
    network_mode: host
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "3"
```

### 启动node-exporter容器

```shell
# docker-compose up -d node-exporter
```

### 访问地址

```
http://10.0.249.177:9100/metrics
```

# nginx-vts-exporter

## 安装包方式

### 前置条件

nginx编译时需加入nginx_vts_module。可以通过http://localhost:80/http_status/format/json访问vts页面

### 下载

-[下载页](https://github.com/hnlq715/nginx-vts-exporter/archive/v0.5.zip)

### 安装

进入网关服务器，执行命令：
```shell
$ wget -O nginx-vts-exporter-0.5.zip https://github.com/hnlq715/nginx-vts-exporter/archive/v0.5.zip
$ unzip nginx-vts-exporter-0.5.zip
$ mv nginx-vts-exporter-0.5  /usr/local/prometheus/nginx-vts-exporter
$ chmod +x /usr/local/prometheus/nginx-vts-exporter/bin/nginx-vts-exporter
```
### 启动

进入网关服务器，执行命令：
```shell
$ cd /usr/local/prometheus/nginx-vts-exporter/bin/
$ ./nginx-vts-exporter  -nginx.scrape_uri http://localhost:61101/http_status/format/json &
```

### 访问地址

```
http://10.0.249.177:9913/metrics
```