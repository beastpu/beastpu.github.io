---
layout: post
title: 限制ip访问频率
category: nginx
tags: [limit ip rate]
---

# 限制ip访问频率

限制ip访问频率

### ngx\_http\_limit\_req\_module

nginx的ngx\_http\_limit\_req\_module模块提供了单个ip限制访问次数的功能，此模块默认安装。配置如下：

```
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

    ...

     server {
        limit_req zone=one;
        listen       9006 default_server;
        listen       [::]:9006 default_server;
	      server_name  _;
        root         /usr/share/nginx/html;

	      # Load configuration files for the default server block

        location / {
        }

```

`limit\_req_zone $binary_remote_addr zone=one:10m rate=1r/s`其中one表示设置的共享内存大小，访问频率为每次一个请求。
`limit_req_zone` 只能配置在http域下，支持http协议。
`limit_req zone=one burst=5;`  可以写在server或location区域内，burst表示最大请求的次数。
我们设定了1秒一个请求的频率，如果一秒多于一个请求且小于5，则多余的请求被延迟，如果大于5，则返回503错误。 如果不希望延迟，则可以配置为  `limit_req zone=one burst=5 nodelay;`                                                                                                                                                                                                                                                                                                                            

可以用ab工具并发测试，测试结果如下：

```text
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 200 145 "-" "ApacheBench/2.3" "-"
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 503 3693 "-" "ApacheBench/2.3" "-"
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 503 3693 "-" "ApacheBench/2.3" "-"
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 503 3693 "-" "ApacheBench/2.3" "-"
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 503 3693 "-" "ApacheBench/2.3" "-"
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 503 3693 "-" "ApacheBench/2.3" "-"
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 503 3693 "-" "ApacheBench/2.3" "-"
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 503 3693 "-" "ApacheBench/2.3" "-"
::1 - - [02/Dec/2019:22:40:51 +0900] "GET /a.html HTTP/1.0" 503 3693 "-" "ApacheBench/2.3" "-"

```

> 注意访问的location地址必须返回为html页面，访问返回字符串的location不生效。

