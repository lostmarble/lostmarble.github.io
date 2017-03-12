---
layout: post
title: 2015-09-22-nginx-squid-nginx
category: 技术 
tags: Essay
keywords: MongoDB,总结,2015
description: 
---

工作中使用nginx进行负载均衡与代理服务功能，因为中间加了一个squid，

整体的架构如下：

Nginx1-->Squid-->Nginx2 

故需要配置一下：
Nginx1.conf

```
worker_processes  32;

#unlimit -n 65535
worker_rlimit_nofile 65535;

error_log  logs/error.log  error;
pid        logs/nginx.pid;


events {
       use epoll;
       worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;


    log_format main
            '$remote_user [$time_local]  $http_x_Forwarded_for $remote_addr  $request '
            '$http_x_forwarded_for '
            '$upstream_addr '
            'ups_resp_time: $upstream_response_time '
            'request_time: $request_time'

    access_log  logs/access.log  main;

    sendfile        on;
    tcp_nopush      on;

    keepalive_timeout  65;
    server_tokens   off;

    gzip  on;
    gzip_vary  on;

    upstream web_app{
    	     server 127.0.0.1:3128
	     }

    server {
            listen       2195;
	    server_name  hostname;
	    access_log  logs/access.log  main;


	    location /nginxstatus{
	    stub_status on;
	    access_log on;
	    }
	    error_page   500 502 503 504  /50x.html;
	    location = /50x.html {
	    root   html;
	    }

        location / {
        proxy_set_header Host $http_host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header readhost $http_host;
	proxy_pass http://web_app;
	}
    }
}
    
```

squid.conf

```
http_port 2195 accel vhost vport
cache_mem 64 MB
maximum_object_size 4 MB
cache_dir ufs /var/spool/squid 100 16 256
access_log /var/log/squid/access.log
cache_peer spiderm.chinacloudapp.cn parent 2195 0 no-query originserver
http_access allow all
visible_hostname squid.cib.com.cn
cache_mgr xuzuoxin@cib.com.cn
```


nginx2.conf
```
#user  nginx nginx;
worker_processes  1;

#unlimit -n 65535
worker_rlimit_nofile 65535;

error_log  logs/error.log  error;
pid        logs/nginx.pid;

events {
    #use epoll;
        worker_connections  1024;
	}

# 允许尽可能地处理更多的连接数，如果worker_connections配置太低，会产生大量无效连接请求
#multi_accept on;


# 缓存高频操作文件的FDs（文件描述符/文件句柄）
# 在我的设备环境中，通过修改以下配置，性能从 560k 请求/秒 提升到 904k 请求/秒。
# 我建议你对以下配置尝试不同的组合，而不是直接使用这几个数据。
#open_file_cache max=200000 inactive=20s;
#open_file_cache_valid 30s;
#open_file_cache_min_uses 2;
#open_file_cache_errors on;

http {
    include       mime.types;
        default_type  application/octet-stream;

#    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
#                      '$status $body_bytes_sent "$http_referer" '
#                     '"$http_user_agent" "$http_x_forwarded_for"';

log_format main
 '$remote_user [$time_local]  $http_x_Forwarded_for $remote_addr  $request '
  '$http_x_forwarded_for '
   '$upstream_addr '
    'ups_resp_time: $upstream_response_time '
     'request_time: $request_time'

    access_log  logs/access.log  main;

    sendfile        on;
        tcp_nopush      on;
	   # tcp_nodely      on;

    keepalive_timeout  65;
        server_tokens   off;

    gzip  on;
        gzip_vary  on;

    server {
            listen       2196;
	            server_name  hostname;
		            resolver 127.0.0.1;
			            #charset utf-8;
				            access_log  logs/access.log  main;
					           # add_header Vary ff-bb-1;

        location / {

            proxy_set_header Host  $http_host;
	    proxy_set_header X-Real-IP $remote_addr;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_pass http://$http_host$uri$is_args$args;
						        }

        location /nginxstatus {
	           stub_status on;
		   access_log on;
	}

        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
		            root   html;
		}
	}
}					
```




