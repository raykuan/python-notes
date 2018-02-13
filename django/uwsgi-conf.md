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

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8090;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        # location / {
        #    root   html;
        #    index  index.html index.htm;
        # }
    }

    server {
        listen       4433 ssl;
        server_name  localhost;
        ssl on;
        ssl_certificate /home/msad/nginx_cert/ysepay.pem;
        ssl_certificate_key /home/msad/nginx_cert/ysepay.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        access_log  logs/access.log;
        error_log  logs/uwsgi_error.log  notice;

        location / {
            root /home/msad/adauth;
            uwsgi_pass 127.0.0.1:6000;
            include uwsgi_params;
        }

        location /static {
            # root /home/adself/msldap;
            alias /home/msad/adauth/static;
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
