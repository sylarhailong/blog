---
title: nginx配置详解
date: 2017-08-17 20:15:54
categories: nginx
tags: nginx.conf
---
nginx是一个很强大的高性能Web和反向代理服务器，每次修改nginx配置都需要重新整理查找，因此总结了nginx配置信息，便于以后查阅.
本文主要介绍nginx.conf的配置，与具体业务相关的配置不包含在此部分内容中。
# nginx.conf的例子
```
#nginx用户组
user  nobody nobody; 

#nginx工作进程数
worker_processes  8;
 
#nginx可以使用gdb进行调试,Crash的时候产生Core文件
#working_directory表示Core文件存放的目录
#worker_rlimit_core表示单个worker子进程所使用的Core文件大小的最大值
working_directory /test/nginx;
worker_rlimit_core 2G;

#错误日志
error_log  logs/error.log error;

pid     sbin/nginx.pid;

#指定进程可以打开的最大描述符：数目。
#这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除
#但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。

#现在在Linux 2.6内核下开启文件打开数为65535，worker_rlimit_nofile就相应应该填写65535。

#这是因为nginx调度时分配请求到进程并不是那么的均衡
#所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
worker_rlimit_nofile 51200;

events
{
        use epoll; #使用epoll的I/O 模型，感兴趣的可以仔细了解下每个I/O模型的区别 不同操作系统下配置不同
        worker_connections  51200; #每个工作进程的最大连接数量 理论上每台nginx服务器的最大连接数为：worker_processes*worker_connections
}

http
{
	#nginx通过服务器端文件的后缀名来判断这个文件属于什么类型，再将该数据类型写入HTTP头部的Content-Type字段中，发送给客户端
        include    mime.types;
	#nginx默认返回的Content-Type字段
        default_type  application/octet-stream;
	
	#set_real_ip_from指令是告诉nginx，10.0.0.0是我们的反代服务器，不是真实的用户IP，可以包含多行这个配置
        set_real_ip_from   10.0.0.0/8;
        set_real_ip_from   192.168.0.0/16;

	#当real_ip_recursive为off时，nginx会把real_ip_header指定的HTTP头中的最后一个IP当成真实IP
	#当real_ip_recursive为on时，nginx会把real_ip_header指定的HTTP头中的最后一个不是信任服务器的IP当成真实IP
	#当用户访问到我们的网站过程中经过多个路由转发，所以X-Forwarded-For应该是101.10.33.11, 192.168.0.10，
	#如果设置成off，取到的ip是192.168.0.10，如果设置成on，获得的ip为101.10.33.11，把取得的这个ip当成用户真实的ip
        real_ip_recursive on;

	#real_ip_header则是告诉nginx真正的用户IP是存在X-Forwarded-For请求头中
        real_ip_header     X-Forwarded-For;
        include xnet.conf;

	#日志格式
        log_format main '$host $remote_addr [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
                        '$ip2loc_code4 $cookie_SUID $request_time $cookie_SSUID '
                        '$http_x_forwarded_for $request_length $cookie_YYID ';
        log_format mini '$time_local $status $body_bytes_sent $request_time $server_name $status $request_uri';
        #访问日志打印地址
	access_log "logs/${server_name}_access_log" main;
        access_log logs/status_log mini;

	#给站点配置多个别名的时候需要配置该选项
        server_names_hash_bucket_size 128;

	#指定客户端请求的http头部缓冲区大小绝大多数情况下一个头部请求的大小不会大于1k不过如果有
	#来自于wap客户端的较大的cookie它可能会大于1k，nginx将分配给它一个更大的缓冲区
	#这个值可以在large_client_header_buffers里面设置。
	#这两个配置只针对get请求，post请求的配置可自行查阅
        client_header_buffer_size 32k;
        large_client_header_buffers 4 32k;

	#允许客户端请求的最大单文件字节数。如果有上传较大文件，需要设置，防止返回给用户返回413
        client_max_body_size 20m;

	#开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，减少用户空间到内核空间的上下文切换。
	#对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载
        sendfile        on;
        
	#在Linux上使用TCP_CORK套接字选项,关于此字段和下个字段的详解看文后的链接文章
	tcp_nopush      on;
        #发送应立即发出的短消息
	tcp_nodelay on;
	#client_header_buffer_size用于指定响应客户端的超时时间。
	#这个超时仅限于两个连接活动之间的时间，如果超过这个时间，客户端没有任何活动，Nginx将会关闭连接。
        send_timeout       300;
	
	#以下两个配置都是http_proxy模块的配置
	#nginx跟后端服务器连接超时时间(代理连接超时)
        proxy_connect_timeout 2;
	#连接成功后，与后端服务器两个成功的响应操作之间超时时间(代理接收超时)
        proxy_read_timeout 5;

	#nginx连接resin采用hmux协议 提高效率
        hmux_temp_path /dev/shm/nginx/hmux_temp/;

        hmux_buffers 4 16k;
        hmux_connect_timeout 2s;
        hmux_read_timeout 7s;
        hmux_send_timeout 6s;
        hmux_buffer_size 16k;
        hmux_flush always;
	
	#定义路径下默认访问的文件名，一般跟着root放
        index index.html index.htm index.shtml;
	
	#设置允许压缩的页面最小字节数，页面字节数从header头得content-length中进行获取。默认值是20。
	#建议设置成大于1k的字节数，小于1k可能会越压越大
        gzip_min_length  1000;

	#gzip压缩比，1压缩比最小处理速度最快，9压缩比最大但处理速度最慢(传输快但比较消耗cpu)
        gzip_comp_level  6;
	
	#匹配mime类型进行压缩，无论是否指定,”text/html”类型总是会被压缩的。
        gzip_types       text/plain text/xml application/x-javascript text/css application/xml;

	#该命令的作用是显示或隐藏掉版本,出现错误的时候看不到nginx的版本信息
        server_tokens off;
	#业务相关的配置独立出来
        include vhosts/*.conf;
}
```
# 整理过程中参考有用的文章
* [nginx服务器安装及配置文件详解](https://segmentfault.com/a/1190000002797601)
* [TCP_NODELAY 和 TCP_NOPUSH](http://xiaomaimai.blog.51cto.com/1182965/1557773)
* [The module implements resin's hmux protocol in nginx](https://github.com/wangbin579/nginx-hmux-module)
* [Nginx配置文件（nginx.conf）配置详解](http://blog.csdn.net/tjcyjd/article/details/50695922)

