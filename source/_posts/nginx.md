---
title: CentOs7配置Nginx的SSl
categories: 
- hexo相关
tags:
- 博客
- hexo折腾
---



### 使用的设备、技术

- 设备:[腾讯云服务器](https://cloud.tencent.com/act)
- 技术:nginx 1.14.1

事先在腾讯云申请好[SSL](https://console.cloud.tencent.com/certoverview)证书（免费），以及[域名](https://console.cloud.tencent.com/domain/all-domain)（便宜的）。等待证书申请完毕后，下载好Nginx的文件

- xxxxxx_bundle.crt 
- xxxxxx_bundle.pem (无用)
- xxxxxx.csr    （无用）
- xxxxxx.key

去腾讯云控制台自带[WebShell](https://workbench.cloud.tencent.com/webshell_lighthouse)上传**xxxxxx_bundle.crt**、**xxxxxx.key**。默认会上传到/root这个路径下面

```bash
$ mv ./xxxxxx_bundle.crt /etc/pki/nginx/xxxxxx_bundle
$ mv ./xxxxxx_bundle.crt /etc/pki/nginx/xxxxxx.key
```
移动文件到这个路径下，然后修改nginx.conf文件

```bash
$ vim /etc/nginx/nginx.conf
```
添加一个
```
     server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  *.xxxxxx.cn;
        root         /usr/share/nginx/xxxx;

        ssl_certificate "/etc/pki/nginx/xxxxxx_bundle";
        ssl_certificate_key "/etc/pki/nginx/xxxxxx.key";

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

```
然后把原本的
``` bash
        server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  *.xxxxxx.cn;
      #  root         /usr/share/nginx/xxxxx;

        # Load configuration files for the default server block.
       # include /etc/nginx/default.d/*.conf;

      #  location / {
      #  }

      #  error_page 404 /404.html;
      #      location = /404.html {
      #  }
        return 301 https://$host$request_uri;
      #  error_page 500 502 503 504 /50x.html;
      #      location = /50x.html {
      #  }
    }
```
我这里直接return了301把http请求重定向到https类型下面，如果需要保留http请求类型则不需要这一行，把其他的注视放开即可