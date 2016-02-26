---
layout: post
title: Nginx双层代理负载均衡的会话保持功能
category: 技术 
keywords: nginx,sticky
---


## 需求 
工作中有个项目需要使用了Nginx做代理服务器，考虑到负载均衡，使用另外一个Nginx做请求分发，具体的风络拓扑如下:


                             --->Nginx代理服务器--->互联网
                             |
应用---->Nginx负载均衡服务器---->Nginx代理服务器--->互联网
                             |
                             --->Nginx代理服务器--->互联网


由于应用与互联网之间有会话(session)，使用多个代理容易导致会话丢失，查了下Nginx的官方文档，建议使用sticky指令。
sticky在负载均衡服务器与应用之间添加了一个cookie来跟踪会话(默认是route，可以配置)，它建立了互联网会话与负载均衡会话的一个映射。即两个cookie值之间的映射。
下面通过实验来验证下这个功能。

## 解决方案 
下载Nginx-sticky-module:
https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng

重新编译Nginx


```
    ./configure ... --add-module=/absolute/path/to/nginx-sticky-module-ng
     make
     make install

```
在负载均衡的Nginx配置文件中添加sticky指令, 其中负载均衡服务器部署到127.0.0.1:2200, 两个代理服务器部署到127.0.0.1:9000,127.0.0.1:9001

```
upstream {
    sticky;
    server 127.0.0.1:9000;
    server 127.0.0.1:9001;
}
```

使用webpy写了一个基本的web服务,部署到127.0.0.1:2233


```
import web

urls = (
    '/', 'index',
    '/wc', 'wc' #with cookie
)

class index:
    def GET(self):
        i = web.input(age='25')
        web.setcookie('age', i.age, 3600)
        return "Hello, world!"
class wc:
    def GET(self):
        return web.cookies().get('age')

if __name__ == "__main__":
    app = web.application(urls, globals())
    app.run()
```

## 测试结果
通过代理来访问该服务：

```
curl -v 127.0.0.1:2233 -x 127.0.0.1:2200

HTTP/1.1 502 Bad Gateway
Server: nginx
Date: Wed, 29 Jul 2015 07:32:18 GMT
Content-Type: text/html
Content-Length: 537
Connection: keep-alive
Set-Cookie: route=6ae1d9c528951fb4953900fd374f2cec; Path=/


curl -v 127.0.0.1:2233 -x 127.0.0.1:2200

HTTP/1.1 200 OK
Server: nginx
Date: Wed, 29 Jul 2015 07:33:17 GMT
Transfer-Encoding: chunked
Connection: keep-alive
Set-Cookie: route=8170b2a547907f7995e93d8f569b50f2; Path=/

```
可见，连续两个请求均未携带cookie时，会被负载均衡到两个代理服务器

下面将cookie保存到文件cookie.txt

```
curl -v 127.0.0.1:2233/ -x 127.0.0.1:2200 -c cookie.txt

HTTP/1.1 200 OK
Server: nginx
Date: Wed, 29 Jul 2015 07:36:26 GMT
Transfer-Encoding: chunked
Connection: keep-alive
Added cookie route="8170b2a547907f7995e93d8f569b50f2" for domain 127.0.0.1, path /, expire 0
Set-Cookie: route=8170b2a547907f7995e93d8f569b50f2; Path=/

```

在请求中携带cookie
```
curl -v 127.0.0.1:2233/wc -x 127.0.0.1:2200 -b cookie.txt
```

多次运行上述命令，返回结果如下, Cookie均为route=8170b2a547907f7995e93d8f569b50f2, 可见对于同一个应用层cookie Nginx均会把它发送到同一个代理服务器。
```
GET http://10.11.4.4:2233/wc HTTP/1.1
User-Agent: curl/7.19.7 (x86_64-redhat-linux-gnu) libcurl/7.19.7 NSS/3.16.2.3 Basic ECC zlib/1.2.3 libidn/1.18 libssh2/1.4.2
Host: 10.11.4.4:2233
Accept: */*
Proxy-Connection: Keep-Alive
Cookie: route=8170b2a547907f7995e93d8f569b50f2
```


