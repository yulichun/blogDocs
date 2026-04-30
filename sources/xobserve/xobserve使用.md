## 主要地址

- 官方文档：https://zh.datav.io/docs
- 库地址：https://github.com/data-observe/datav
- 演示地址：https://play.datav.io/

## 开发部署

### 下载源码包

https://github.com/data-observe/datav

### 后端开发


1. datav分前后端，后端代码在主目录的query目录下，基于go开发环境。
2. 需要cgo方式编译，需要gcc。
   a. windows需要下载minGw
   b.linux需要gcc：

   ```
yum install gcc
yum install gcc-c++ libstdc++-devel
 
yum install glibc-static.x86_64 -y
   ```

3. 设置使用cgo编译：

```
go env set CGO_ENABLED=1
```

4. 进入query目录，编译：
windows

```
go build -o datav.exe 
```

linux

```
go build -o datav
```

5. 启动：
windows

```
./datav.exe 
```

linux

```
./datav
```


### 前端开发

1. 前端代码在主目录的ui目录下，基于node 18以上版本（我用的18.18.0）+ yarn开发环境。
2. 进入ui目录，下载依赖：

```
yarn install
```

3. 配置后端地址，在.env目录下指定后端地址:

```
VITE_API_SERVER_DEV=http://localhost:10086
VITE_API_SERVER_PROD=https://api.datav.io 
```

4. 使用vite启动开发模式：
vite会在下载依赖时下载到node_modules目录

```
.\node_modules\.bin\vite
```

5. 访问http://localhost:5173