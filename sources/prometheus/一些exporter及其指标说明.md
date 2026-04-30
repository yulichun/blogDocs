# nginx指标

存在3种nginx的exporter

## nginx_vts_exporter

> - 最优的nginx exporter插件
> - 依赖于nginx-vts模块的实现
> - [源码地址](https://github.com/hnlq715/nginx-vts-exporter)
> - [nginx_vts_module数据样例,访问http://ip:8080/status](./exporter_example/nginx_vts_example.txt)
> - [数据样例](./exporter_example/nginx_vts_exporter_example.txt)

| metric  |	注释 | 数据类型 |	tag标签 | tag标签说明 |
| ------------ | -------- | -------- | -------- | ---- |
| nginx_server_connections | 	连接数 | 	gauge | | |
| nginx_server_requestMsec | 	平均响应时间 |	gauge | | | 
| nginx_server_requests | 	各种状态码响应总数 | 	counter |	code [2xx, 3xx, 4xx, 5xx, total], host (or domain name) | |
| nginx_server_bytes | 	服务流量字节数 | 	counter | 	direction [in, out], host (or domain name) | 方向：流入、流出 | 
| nginx_server_cache | 	服务缓存 | 	counter | 	status [bypass, expired, hit, miss, revalidated, scarce, stale, updating], host (or domain name) |	状态：bypass, expired, hit, miss, revalidated, scarce, stale, updating |
| nginx_filter_requests | 	经过过滤模块的请求数 | 	counter | code [2xx, 3xx, 4xx, 5xx and total], filter, filter name | | 
| nginx_filter_bytes | 	经过过滤模块的流量字节数 | 	counter | 	direction [in, out], filter, filter name | | 
| nginx_filter_responseMsec | 	经过过滤模块的响应时间 | 	gauge |	filter, filter name | | 
| nginx_upstream_requestMsec | 	各负载均衡模块平均响应时间 | 	gauge |	code [2xx, 3xx, 4xx, 5xx and total], upstream (or upstream name) | | 
| nginx_upstream_requests |	各负载均衡模块各个状态码总数 |	counter |	direction [in, out], upstream (or upstream name) | | 
| nginx_upstream_bytes | 	各负载均衡模块流量字节数 | 	counter |	backend (or server), in_bytes, out_bytes, upstream (or upstream name) | | 

## hnlq7_nginx_prometheus_exporter

> - 基于openresty的开发的prometheus exporter实现
> - [源码地址](https://github.com/hnlq715/nginx-prometheus-metrics)
> - [数据样例](./exporter_example/hnlq715_nginx_prometheus_example.txt)

| metric | 	注释 |	数据类型 |
| ------------ | -------- | -------- |
| nginx_http_connections | 	http连接数 | 	gauge | 
| nginx_http_request_time_bucket | 	处在各种请求时长区间中的请求数	histogram |
| nginx_http_requests | 	请求总数	counter |
| nginx_http_upstream_connect_time_bucket | 	处在各负载均衡模块的连接时长区间中的请求数	histogram |
| nginx_http_upstream_header_time_bucket | 	处在各负载均衡模块的header时长区间中的请求数	histogram |
| nginx_http_upstream_requests	负载均衡模块的总请求数 | 	counter |
| nginx_http_upstream_response_time_bucket | 	处在各负载均衡模块的响应时长区间中的请求数	histogram |

## nginxinc_nginx_exporter

> - a、基于nginx自带的stub_status实现
> - b、基于非开源nginx-plus的实现，监控指标最全
> - [源码地址](https://github.com/nginxinc/nginx-prometheus-exporter)
> - [数据样例](./exporter_example/nginxinc_nginx_prometheus_example.txt)

| metric | 	注释 |	数据类型 |
| ------------ | -------- | -------- |
| nginx_connections_accepted |	接收的连接数 |	counter |
| nginx_connections_active |	活跃的连接数 |	gauge |
| nginx_connections_handled |	已处理的连接数	 |counter |
| nginx_connections_reading |	nginx正在读取请求头的连接数 |	gauge |
| nginx_connections_waiting |	空闲的连接数 |	gauge |
| nginx_connections_writing |	nginx正在往客户端写数据的连接数 |	gauge |
| nginx_http_requests_total |	所有请求数 |	counter |
| nginx_up |	已启动的nginx |	gauge |

# jvm-exporter

| metric | 	注释 |	数据类型 |
| ------------ | -------- | -------- |
| process_virtual_memory_bytes |	虚拟内存大小(以字节为单位 |	gauge |
| process_resident_memory_bytes |	常驻内存大小(以字节为单位) |	gauge |
| jvm_threads_current |	JVM的当前线程数 |	gauge |
| jvm_threads_daemon |	JVM的守护进程线程数 |	gauge |
| jvm_threads_peak |	JVM的峰值线程数 |	gauge |
| jvm_threads_started_total |	JVM启动的线程总数 |	counter |
| jvm_memory_bytes_used |	给定JVM内存区域的已使用的字节 |	gauge |
| jvm_memory_bytes_committed |	给定JVM内存区域的已提交的字节 |	gauge |
| jvm_memory_bytes_max |	给定JVM内存区域的最大字节 |	gauge |
| jvm_memory_bytes_init |	给定JVM内存区域的初始化字节 |	gauge |
| jvm_memory_pool_bytes_used |	给定JVM缓存池已使用的字节。 |	gauge |
| jvm_memory_pool_bytes_committed |	给定JVM缓存池已提交的字节。 |	gauge |
| jvm_memory_pool_bytes_max |	给定JVM缓存池最大字节。 |	gauge |
| jvm_memory_pool_bytes_init |	给定JVM缓冲池初始字节 |	gauge |
| jvm_threads_deadlocked |	处于死锁状态的jvm线程，等待获取对象监视器或可拥有的同步器 |	gauge |
| jvm_threads_deadlocked_monitor |	处于死锁状态的jvm线程，等待获取对象监视器 |	gauge |
| jvm_classes_loaded_total |	自JVM开始执行以来加载的类的总数 |	counter |
| jmx_config_reload_success_total |	成功重新加载配置的次数。 |	counter |
| jvm_buffer_pool_used_bytes |	给定JVM缓冲池中已使用的字节。 |	gauge |
| process_cpu_seconds_total |	用户和系统CPU总时间(以秒为单位) |	counter |

# node-exporter

> - [数据样例](./exporter_example/node_exporter_example.txt)

| metric | 	注释 |	数据类型 | tags |
| ------------ | -------- | -------- | -------- |
| node_disk_io_now |	当前正在进行的磁盘I/Os数量。 |	gauge  |	device[sda，sr0，sr1] |
| node_disk_io_time_weighted_seconds_total |  |	 | node_disk_io_time_weighted_seconds_total	counter	device[sda，sr0，sr1] |
| node_disk_read_bytes_total |	磁盘总读取字节数  |	counter  |	device[sda，sr0，sr1] |
| node_disk_read_time_seconds_total |	磁盘总读取秒数  |	counter  |	device[sda，sr0，sr1] |
| node_disk_reads_completed_total |	读取完成数  |	counter |	device[sda，sr0，sr1] |
| node_disk_reads_merged_total |	node_disk_reads_merged_total  |	counter  |	device[sda，sr0，sr1] |
| node_disk_write_time_seconds_total |	磁盘写总耗时秒数 |	counter  |	device[sda，sr0，sr1] |
| node_disk_writes_completed_total |	磁盘写完成数 |	counter |	device[sda，sr0，sr1] |
| node_disk_writes_merged_total |	node_disk_writes_merged_total |	counter |	device[sda，sr0，sr1] |
| node_disk_written_bytes_total |	磁盘写总字节数 |	counter |	device[sda，sr0，sr1] |
| node_cpu_seconds_total | Seconds	cpu在每种模式下花费的时间。 |	counter | |
| node_memory_MemAvailable_bytes |	正在使用的内存   5.865127936e+09 |	gauge | |
| node_memory_MemFree_bytes |	空闲内存  5.241372672e+09 |	gauge | |
| node_memory_MemTotal_bytes |	总内存  8.20297728e+09 |	gauge | |
| node_memory_Buffers_bytes |	缓冲字节内存信息  1.29024e+06 |	gauge | |
| node_memory_Cached_bytes |	缓存字节内存信息   7.56338688e+08 |	gauge | |
| node_cpu_guest_seconds_total |	用户进程使用的cpu总秒数 |	counter |	cpu,mode["nice","user"] |
| node_disk_io_time_seconds_total |	花在磁盘的I/Os上的总秒数。 |	counter |	device[sda，sr0，sr1] |
| node_schedstat_waiting_seconds_total |	cpu处理waiting状态花的总秒数 |	counter |	cpu |
| node_schedstat_running_seconds_total |	cpu花在进程上的总秒数 |	counter |	cpu |
| node_cpu_seconds_total |	cpu总秒数 |	counter |	cpu，mode[idle，iowait，irq，nice，softirq，steal，system，user] |
| node_filesystem_avail_bytes |	文件系统可用字节数 |	gauge |	device[/dev/sda2,lxcfs,tmpfs],fstype[ext4,fuse.lxcfs,tmpfs],mountpoint[/,/var/lib/lxcfs] |
| node_filesystem_device_error |	磁盘设备读取错误数 |	gauge |	device[/dev/sda2,lxcfs,tmpfs],fstype[ext4,fuse.lxcfs,tmpfs],mountpoint[/,/var/lib/lxcfs] |
| node_filesystem_files |	文件数 |	gauge |	device[/dev/sda2,lxcfs,tmpfs],fstype[ext4,fuse.lxcfs,tmpfs],mountpoint[/,/var/lib/lxcfs] |
| node_filesystem_files_free |	自由文件数 |	gauge |	device[/dev/sda2,lxcfs,tmpfs],fstype[ext4,fuse.lxcfs,tmpfs],mountpoint[/,/var/lib/lxcfs] |
| node_filesystem_free_bytes |	以字节为单位的文件系统空闲空间 |	gauge |	device[/dev/sda2,lxcfs,tmpfs],fstype[ext4,fuse.lxcfs,tmpfs],mountpoint[/,/var/lib/lxcfs] |
| node_filesystem_readonly |	只读状态 |	gauge |	device[/dev/sda2,lxcfs,tmpfs],fstype[ext4,fuse.lxcfs,tmpfs],mountpoint[/,/var/lib/lxcfs] |
| node_filesystem_size_bytes |	文件系统大小 |	gauge |	device[/dev/sda2,lxcfs,tmpfs],fstype[ext4,fuse.lxcfs,tmpfs],mountpoint[/,/var/lib/lxcfs] |
| node_network_address_assign_type |	地址分配类型数 |	gauge |	device[docker0,ens33,lo,tun0] |
| node_network_carrier |	载波值 |	gauge |	device[docker0,ens33,lo,tun0] |
| node_network_carrier_changes_total |	载波变动总大小 |	gauge |	device[docker0,ens33,lo,tun0] |
| node_network_carrier_down_changes_total |	 |	counter |	device[docker0,ens33,lo,tun0] |
| node_network_carrier_up_changes_total	 |  |	counter |	device[docker0,ens33,lo,tun0] |
| node_network_device_id |	 |	gauge |	device[docker0,ens33,lo,tun0] |
| node_network_mtu_bytes |	 mtu字节数 |	gauge |	device[docker0,ens33,lo,tun0] |
| node_network_name_assign_type |	 |  |		device[docker0,ens33,lo,tun0] |
| node_netstat_Icmp_InErrors |	Icmp错误统计 |	untyped | |
| node_netstat_Icmp_InMsgs |	IcmpInMsgs 统计 |	untyped | |
| node_netstat_Icmp_OutMsgs |	IcmpOutMsgs 统计	 |untyped | |
| node_netstat_Tcp_ActiveOpens |	Tcp开启次数 |	untyped | |
| node_netstat_Tcp_CurrEstab |	当前tcp连接建立数	 | untyped | |
| node_netstat_Tcp_InErrs |	Tcp连接错误数量 |	untyped |  |
| node_network_receive_packets_total |	网络设备接收数据包总数 |	counter |  |
| node_network_receive_drop_total |	丢包数	counter	 |  |
