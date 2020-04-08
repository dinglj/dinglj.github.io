---
layout : post
title : "ngnix安装"
category : ngnix
duoshuo: true
date : 2020-03-31
tags : [ngnix,安装]
---
参考文章: https://www.cnblogs.com/tangjiansheng/p/6928722.html
默认安装的是1.12.2版本
shell命令
yum install nginx
systemctl start nginx

#反向代理配置
#host根据情况修改
u pstream a.com { 
      ip_hash; #负载均衡算法,保持session
      server  192.168.5.126:80; #负载均衡地址1
      server  192.168.5.27:80; #负载均衡地址2
} 
  
server{ 
    listen 80; 
    server_name a.com; #host根据情况修改
    location / { 
        proxy_pass         http://a.com; #host根据情况修改
        proxy_set_header   Host             $host; 
        proxy_set_header   X-Real-IP        $remote_addr; #判断负载均衡地址
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for; 
    } 
}

#前端配置
server{ 
    listen 80; 
    server_name b.com; #host根据情况修改
    index index.html; #默认打开页面
    root /data0/htdocs/www; #映射到本地的目录
}

#性能调优(简单)
#worker_processes 8; #设置worker的进程数
#worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000; #进程和cpu绑定
#worker_rlimit_nofile 65535; #进程打开的文件句柄数
events
{
   use epoll; #使用epoll模型
   worker_connections 65535;#设置连接数；最大连接数=worker_processes * worker_connections。
}
http{
charset utf-8;#编码设置
server_tokens off; #隐藏http中nginx信息,防止攻击
  gzip on;#gzip 压缩减少流量
sendfile on; #从磁盘直接发送文件,避免内存拷贝
    tcp_nopush on; #数据包批量发送
tcp_nodelay on; #数据包是否延迟发送(考虑小数据快速响应)
keepalive_timeout 60; #连接复用,keepalive http协议的最长时间
###本地静态文件的读取影响###########
open_file_cache max=204800 inactive=20s; #可以考虑设置
    open_file_cache_min_uses 1; #可以考虑设置
    open_file_cache_valid 30s;  #可以考虑设置
####FastCGI参数优化,应该对本地静态文件的读取有影响，对使用fastcgi的程序有影响，比如php,暂不评价####
fastcgi_cache_path /usr/local/nginx/fastcgi_cache levels=1:2
                 keys_zone=TEST:10m
                 inactive=5m;
   fastcgi_connect_timeout 300; #可以考虑设置
   fastcgi_send_timeout 300; #可以考虑设置
   fastcgi_read_timeout 300; #可以考虑设置
   fastcgi_buffer_size 16k;
   fastcgi_buffers 16 16k;
   fastcgi_busy_buffers_size 16k;
   fastcgi_temp_file_write_size 16k;
   fastcgi_cache TEST; #可以考虑设置
   fastcgi_cache_valid 200 302 1h;
   fastcgi_cache_valid 301 1d;
   fastcgi_cache_valid any 1m;
   fastcgi_cache_min_uses 1; #可以考虑设置
   fastcgi_cache_use_stale error timeout invalid_header http_500; 
}

#防攻击优化（备用）
###(大文件)流量攻击#####
client_body_buffer_size  1K;
client_header_buffer_size 1k;
client_max_body_size 1k;
large_client_header_buffers 2 1k;
####大量连接保持导致的tcp超时攻击######
client_body_timeout   10;
client_header_timeout 10;
keepalive_timeout     65;
send_timeout          10;
#####单个IP大量连接的tcp攻击#######
limit_zone slimits $binary_remote_addr 5m;
limit_conn slimits 5;

#其他优化(对于tcp底层三次握手的攻击防范)
可以考虑linux操作系统层面的tcp优化（/etc/sysctl.conf）
