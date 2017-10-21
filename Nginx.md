## Nginx
@(Linux)[nginx]
#### 1.Nginx常用命令：
```bash
nginx -v #查看nginx版本
nginx -t #检查配置文件nginx.conf的正确性
nginx #启动nginx
nginx -s reload #重新载入配置文件
nginx -s reopen #重启nginx
nginx -s stop #停止nginx
```

#### 2.使用nginx实现动静资源分离：
项目中的静态资源如：js,css,png,mp3,swf等资源文件，通过location对请求url进行匹配：
```bash
location ~ .*\.(js|css|png|gif|jpg|swf|mp3)$ {
    root /data;
    expires 30d;
}
#例如：静态资源路径：localhost:8080/qacvr/static/img/top_logo.png，上面location对该资源拦截后会访问资源/data/qacvr/static/img/top_logo.png
#说明：/data目录下的文件是软连接的目录，这样做是为了不修改项目代码
ln -s /usr/soft/apache-tomcat-7.0.70/webapps/qacvr/static /data/qacvr/static
```

一个示例：
```vim
user www-data;
pid /var/run/nginx.pid;
worker_processes auto;
worker_rlimit_nofile 100000;

events {
    worker_connections 2048;
    multi_accept on;
    use epoll;
}

http {
    server_tokens off;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    access_log off;
    error_log /var/log/nginx/error.log crit;

    keepalive_timeout 10;
    client_header_timeout 10;
    client_body_timeout 10;
    reset_timedout_connection on;
    send_timeout 10;

    limit_conn_zone $binary_remote_addr zone=addr:5m;
    limit_conn addr 100;

    include /etc/nginx/mime.types;
    default_type text/html;
    charset UTF-8;

    gzip on;
    gzip_disable "msie6";
    gzip_proxied any;
    gzip_min_length 1000;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    open_file_cache max=100000 inactive=20s; 
    open_file_cache_valid 30s; 
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

#### 3.gzip压缩功能
编辑 nginx 的配置文件：
```bash
vim /etc/nginx/nginx.conf
```

在 Gzip Settings 中加入如下设置：
```vim
##
# Gzip Settings
##
gzip on;
gzip_min_length 1k;
gzip_buffers 4 16k;
gzip_comp_level 5;
gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php;
```
gzip配置说明：
> 
1) gzip
语法：gzip on/off
默认值：off
作用域：http, server, location
说明：开启或者关闭 gzip 模块，这里使用 on 表示启动
> 
2) gzip_min_length
语法：gzip_min_length length
默认值：gzip_min_length 0
作用域：http, server, location
说明：设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。默认值是0，不管页面多大都压缩。建议设置成大于1k的字节数，小于1k可能会越压越大。
> 
3) gzip_buffers
语法: gzip_buffers number size
默认值: gzip_buffers 4 4k/8k
作用域: http, server, location
说明：设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。4 16k 代表以 16k 为单位，按照原始数据大小以 16k 为单位的4倍申请内存。
> 
4) gzip_comp_level
语法: gzip_comp_level 1..9
默认值: gzip_comp_level 1
作用域: http, server, location
说明：gzip压缩比，1 压缩比最小处理速度最快，9 压缩比最大但处理最慢（传输快但比较消耗cpu）。这里设置为 5。
> 
5) gzip_types
语法: gzip_types mime-type [mime-type ...]
默认值: gzip_types text/html
作用域: http, server, location
说明：匹配MIME类型进行压缩，（无论是否指定）"text/html" 类型总是会被压缩的。这里设置为 text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php。


#### Nginx源码安装
``` bash
$ cd /usr/local/
$ wget http://nginx.org/download/nginx-1.8.0.tar.gz
$ tar -zxvf nginx-1.8.0.tar.gz
$ cd nginx-1.8.0  
$ ./configure --prefix=/usr/local/nginx 
$ make
$ make install
```
#### Systemd管理nginx
``` vim
# /usr/lib/systemd/system/nginx.service
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```


服务器中nginx配置：
```nginx
#user  nobody;
worker_processes  2;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
pid        /run/nginx.pid;


events {
    use epoll;
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    client_max_body_size 300m;
    #Define gzip compression module
    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript;
    
    #Define nginx proxy module
    proxy_http_version 1.1;
    proxy_connect_timeout 5;
    proxy_read_timeout 60;
    proxy_send_timeout 5;
    proxy_buffer_size   16k;
    proxy_buffers     4 64k;
    proxy_busy_buffers_size     128k;
    proxy_temp_file_write_size  128k;
    proxy_headers_hash_max_size 51200;
    proxy_headers_hash_bucket_size 6400;
    #需要在http字段，设置nginx cache
    proxy_temp_path /data/nginx/proxy_temp_path;
    proxy_cache_path /data/nginx/proxy_cache_path levels=1:2 keys_zone=cache_one:50m inactive=20m max_size=30g;
    

    # 设定负载均衡后台服务器列表 
    upstream  backend  { 
        ip_hash; 
        server   192.168.4.40:8080  max_fails=2 fail_timeout=30s ;
    }
                                                   

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
	        proxy_pass        http://backend;  
            proxy_redirect off;
            proxy_cache cache_one;
            proxy_cache_valid 200 30m;
	        # 后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
	        proxy_set_header  Host  $host;
            proxy_set_header  X-Real-IP  $remote_addr;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
 
        }

	    location ~ .*\.(js|css|png|gif|jpg|swf|mp3)$ {
	        root /data;
	        #expires 30d;
	        expires 7d;                     #缓存一天
            proxy_cache cache_one;          #引用定义的缓存模块名称
            proxy_cache_valid 200 1h;       #请求返回值为200的则缓存1小时
            proxy_cache_valid 302 10m;      #请求返回值为301 302的则缓存10分钟
            proxy_cache_valid any 10s;      #其他任何返回值缓存10秒
            add_header X-Via $server_addr;  #定义这个header名为X-Via 通过变量$server_addr明确说明从哪个服务器来响应的 server_addr
            add_header X-Cache-Status $upstream_cache_status; #明确说明是否命中 $upstream_cache_status为upstream模块
	    }   

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```






