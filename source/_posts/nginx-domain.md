---
title: nginx配置多级域名泛解析
date: 2019-05-09 17:49:20
tags: [nginx, domain, 泛解析]
categories:
  - nginx
---

本文主要介绍如何使用nginx配置多级域名，从而实现多级域名的80端口访问同一台服务器的不同端口。

适用场景：
  
  1. 建站服务网站搭建，让你的用户可以通过`[自定义用户名].your-domain.com`访问他们搭建的网站。

  2. 对于我等这些在一台服务器部署了很多web应用又不想让用户在访问的时候加上难看又难记的端口的场景。

PS：关于域名的分级一直是有争议的，在本文中：`.com、.cn、.cc`这些是顶级域名，也就是一级域名。`baidu.com、taobao.com`这些是二级域名。`pan.baidu.com`这些是三级域名，以此类推，N级域名就是在N-1级上追加了一个`xx.`。

<!-- more -->


# 一、泛解析域名

所以，第一步需要做的是：先得把域名解析到服务器，也就是说：首先得保证`a.your-domain.com`和`b.your-domain.com`指向同一服务器ip，然后再考虑不同端口的转发策略。

我这边使用的是阿里云的服务器（假设为`12.34.56.78`），只需要在域名解析页面添加泛域名解析（将`*`解析到`your-domain.com`）即可，这样所有的`*.your-domain.com`就可以都访问到`12.34.56.78`了。

![aliyun-syntax](http://web-site-files.ashshen.cc//blog/nginx-syntax/Screen%20Shot%202019-05-13%20at%2011.23.25%20AM.png)


# 二、nginx配置三级域名解析

经过泛域名解析后，现在`a.your-domain.com`和`b.your-domain.com`都可以访问`12.34.56.78`了。

接下来就是通过nginx配置`12.34.56.78`的`80`端口代理，将`a.your-domain.com`和`b.your-domain.com`分别代理至不同的服务。

nginx配置起来也比较简单，只需要在server解析中做如下配置：

``` nginx
    ...
    server {
        listen       80;
        server_name  localhost, *.your-domain.com;
        client_max_body_size    100m;

        location / {
            if ($host = 'a.your-domain.com')
            {
                proxy_pass https://localhost:6655;
            }
            if ($host = 'b.your-domain.com')
            {
                proxy_pass https://localhost:5566;
            }
            if ($host = 'your-domain.com')
            {
                rewrite ^(.*)$ https://your-domain.com permanent;
            }
        }
    }
    ...

```

上面的配置中，`server_name`配置`*.your-domain.com`表示当前nginx解析的服务域名为泛域名。

然后我们在`location /`的解析中，使用`$host`（用户访问的域名）做判断，根据不同的域名，使用nginx自身的代理语法`proxy_pass`将用户访问转发至不同服务即可。

我这边只是简单的应用转发，所以直接使用`if`语法列出来了，你也可以用通配符来解析。

``` nginx
server {

    listen   80;
    server_name ~^(?<subdomain>.+)\.your-domain\.com$;

    index index.php index.html index.htm;
    set $root_path '/var/www/yanue.net';
    root $root_path;

    try_files $uri $uri/ @rewrite;

    location @rewrite {
        rewrite ^/(.*)$ /index.html?_url=/$1;
    }
}
```

`$1`取到的值即为`subdomain`匹配的值。

配置完成后，使用`nginx -t`测试配置没有语法错误，然后`nginx -s reload`重启nginx即可。


PS：

  1. 如果希望配置四级域名的泛解析（如：`*.fe.your-domain.com`），只需要在域名供应商提供的解析服务中添加`*.fe`解析即可。

  2. 关于域名分级：关于域名的分级一直是有争议的，在本文中：`.com、.cn、.cc`这些是顶级域名，也就是一级域名。`baidu.com、taobao.com`这些是二级域名。`pan.baidu.com`这些是三级域名，以此类推，N级域名就是在N-1级上追加了一个`xx.`






