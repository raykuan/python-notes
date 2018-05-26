## 生产环境安装部署nginx、openssl、uwsgi

#### 1. 安装依赖包（centos7）
```
yum -y install openssl openssl-devel
如果遇到其它的依赖包需要安装，自行查找
```

#### 2. 下载并解压openssl源码包
```
下载地址：https://www.openssl.org/source/
[msad@adapi adauth]$ tar -zxvf /home/msad/software/openssl-1.0.2n.tar.gz -C /home/msad/
```

#### 3. 下载、编译、安装nginx
```
下载地址：https://nginx.org/download/
[msad@adapi adauth]$ tar -zxvf /home/msad/software/nginx-1.12.2.tar.gz -C /home/msad/

如下配置中--with-openssl='第2步中openssl源码包解压目录'
[msad@adapi adauth]$ ./configure --prefix=/home/msad/nginx --with-http_stub_status_module --with-http_ssl_module --with-openssl=/home/msad/openssl-1.0.2n

[msad@adapi adauth]$ make && make install
```

#### 4. 下载、编译、安装nginx
```
下载地址：https://nginx.org/download/
[msad@adapi adauth]$ tar -zxvf /home/msad/software/nginx-1.12.2.tar.gz -C /home/msad/

如下配置中--with-openssl='第2步中openssl源码包解压目录'
[msad@adapi adauth]$ ./configure--prefix=/home/msad/nginx --with-http_stub_status_module --with-http_ssl_module --with-openssl=/home/msad/openssl-1.0.2n

[msad@adapi adauth]$ make && make install
```

#### 5. nginx配置文件（nginx.conf）
```
[msad@adapi adauth]$ cat ~/nginx/conf/nginx.conf
user  wiseo wiseo;
worker_processes  3;
#error_log  logs/error.log;
error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
pid        logs/nginx.pid;
worker_rlimit_nofile 5120;
events {
    worker_connections  5120;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
 
    server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 8 64k;
    client_max_body_size 100m;
    limit_conn_zone $binary_remote_addr zone=one:32k;

    sendfile        on;
    tcp_nopush     on;

    keepalive_timeout  60;
    tcp_nodelay on;

    gzip  on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types       text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    log_format  wwwlogs  '$remote_addr - $remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for';
    
    # include vhost/*.conf;

    #server {
    #    listen 8080;
    #    server_name case.eptok.com;
        # enforce https
    #    return 301 https://$server_name$request_uri;
   # }

    server {
        listen       4443 ssl default;
        #server_name  _;
        server_name  case.eptok.com;
        server_name  _;
        root /home/wiseo/wiseops;
        underscores_in_headers on;
        ssl on;
        ssl_certificate /home/wiseo/ssl_cert/eptok.crt;
        ssl_certificate_key /home/wiseo/ssl_cert/eptok.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        access_log  logs/access.log;
        error_log  logs/mobile_error.log  notice;

        location / {
            # root /home/wiseo/wiseops;
            uwsgi_pass 127.0.0.1:9000;
            include uwsgi_params;
            proxy_connect_timeout 30s;
            proxy_send_timeout   90;
            proxy_read_timeout   90;
            proxy_buffer_size    32k;
            proxy_buffers     4 32k;
            proxy_busy_buffers_size 64k;
            proxy_redirect     off;
            proxy_hide_header  Vary;
            proxy_set_header   Accept-Encoding '';
            proxy_set_header   Host   $host;
            proxy_set_header   Referer $http_referer;
            proxy_set_header   Cookie $http_cookie;
            proxy_set_header   X-Real-IP  $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /static {
            # root /home/wiseo/wiseops/;
            # alias /home/wiseo/wiseops/static;
        }

    }

}

```

#### 6. uwsgi配置文件（uwsgi.ini）
```
[msad@adapi adauth]$ cat uwsgi.ini 
[uwsgi]
socket = 127.0.0.1:6000
master = true
pidfile = /home/msad/nginx/logs/uwsgi.pid
processes = 4
chdir = /home/msad/adauth
wsgi-file = adauth/wsgi.py
profiler = true
enable-threads = true
logdata = true
limit-as = 6048
daemonize = /home/msad/adauth/logs/uwsgi.log
chmod-socket = 664

```
