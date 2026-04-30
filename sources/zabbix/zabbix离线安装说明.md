 
## 1.下载离线安装包

zabbix官网有详细的安装步骤：https://www.zabbix.com/documentation/3.4/zh/manual/installation/install_from_packages。通常选择二进制安装方式，可以在https://www.zabbix.com/download?zabbix=6.0&os_distribution=ubuntu&os_version=18.04&components=server_frontend_agent&db=mysql&ws=apache，在该页面上可以选择具体的版本、系统，并且有详细的安装说明，下文的安装说明就是基于这个页面上的ubuntu18.04的zabbix5.0版本的说明。

先准备一台连线的机器，此处以ubuntu为例，通过apt工具下载zabbix相关的deb包：
 
```
root# wget https://repo.zabbix.com/zabbix/5.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.0-1+bionic_all.deb
root# dpkg -i zabbix-release_5.0-1+bionic_all.deb
root# apt-get update
root# apt install  --download-only zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-agent
root# cd /var/cache/apt/archives
```

deb包将下载到/var/cache/apt/archives，将这些包打包到压缩包，将压缩包上传到离线的机器上，解压压缩包，安装全部deb包：

```
root# dpkg -i *.deb
```

## 2.数据库安装与配置

### 2.1.数据库安装

```
root# sudo apt install mysql-server
root# sudo /etc/init.d/mysql start
```

### 2.2.创建zabbix用户并授权

```
root# mysql -u root -p
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

### 2.3.导入数据库

```
root# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

### 2.4.修改zabbix配置中的数据库密码

```
root# cat /etc/zabbix/zabbix_server.conf |grep -v "^#"|grep DBPassword
```

## 3.修改前端显示时区

```
root# vi /etc/zabbix/apache.conf
php_value max_execution_time 300
php_value memory_limit 128M
php_value post_max_size 16M
php_value upload_max_filesize 2M
php_value max_input_time 300
php_value always_populate_raw_post_data -1
# php_value date.timezone Europe/Riga
# 配置时区
php_value date.timezone Asia/Shanghai
```

## 4.启动所有服务

```
root# systemctl restart zabbix-server zabbix-agent apache2
root# systemctl enable zabbix-server zabbix-agent apache2
```

## 登陆web界面\zabbix初始化

界面地址http://ip/zabbix，如：
http://10.0.247.102/zabbix

登陆后会有环境检查过程，基本直接下一步即可，但是要注意数据库配置步骤，要正确的配置mysql信息。


最后到需要输入用户名密码，输入正确的用户名密码后即可登陆成功：
Admin/zabbix

## 注意事项

1. 如果安装zabbix6.0，要注意zabbix6.0的server模块是基于mysql8.0的，所以在第一步下载离线包前，先更新mysql8.0的deb源（其他系统要做类似的操作），ubuntu18.04参考该文档：
https://blog.csdn.net/qq_42468130/article/details/88595418
