---
layout: post
title: "Django + uwsgi + nginx 配置部署"
data: 2019-05-26 17:12:36 +0800
author: "YY"
tags:
    - 总结
---

## 安装uwsgi

```python
pip install uwsgi
```

## 测试uwsgi运行状态

新建文件test.py

```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"]
```

### 使用uwsgi运行该文件

```python
uwsgi --http :8000 --wsgi-file test.py
```

此语句的意思是，`使用uwsgi运行test.py文件， 采用http模式， 端口8000`

如果端口占用，使用

```python
lsof -i :8000
```

列出占用端口的程序的pid号，并使用以下命令杀掉所有占用端口的程序

```python
sudo kill -9 pid
```

### 访问页面

可以看到hello world 就说明uwsgi运行成功了。



## 安装nginx

```python
sudo apt-get install nginx
```

## 修改nginx配置

文件路径`"/etc/nginx/sites-enabled/default"`

```python
upstream django {
        server 127.0.0.1:8001; #web的socket端口
    }
server {
    listen 80 default_server;
    listen [::]:80 default_server;
 
    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        root /home/ubuntu/dgutpsy; #项目目录
        uwsgi_pass django;
        include /home/ubuntu/dgutpsy/uwsgi_params; #uwsgi_params文件的地址
    }
}
```

完整的uwsgi_params文件内容应该是

```python
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

## nginx 与 uwsgi通信

先使用hello world测试

```python
uwsgi --socket :8001 --wsgi-file test.py
```

访问测试出现hello word，说明nginx 与 uwsgi通信成功!

## 让uwsgi后台运行

### 新建test.ini

```python
[uwsgi]
socket = 127.0.0.1:8001
wsgi-file = /home/ubuntu/dgutpsy/test.py
daemonize = /home/ubuntu/dgutpsy/test.log 
```

```python
uwsgi --ini test.ini
```

其中`daemonize = /home/ubuntu/dgutpsy/test.log`的意思就是后台运行并规定日志输出目录。

## niginx与Django项目通信

### 新建dgutpsy.ini

```python
[uwsgi]
socket = 127.0.0.1:8001
chdir           = /home/ubuntu/dgutpsy
module          = dgutpsy.wsgi
master          = true
processes       = 1 

threads = 2 
max-requests = 6000

daemonize = /home/ubuntu/dgutpsy/run.log
```

然后运行

```python
uwsgi --ini dgutpsy.ini
```