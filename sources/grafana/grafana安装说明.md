1. 拉取grafana的docker镜像

```shell
$ docker pull grafana/grafana:latest
```

2. 创建grafana的数据存储目录,映射到容器的/var/lib/grafana

```shell
$ mkdir /opt/grafana-storage
```

3. 创建prometheus容器

```shell
$ docker run -d -p 3000:3000 --name=grafana -v /opt/grafana-storage:/var/lib/grafana grafana/grafana
```

4. 访问http://{IP}:3000，用户名密码都是admin
