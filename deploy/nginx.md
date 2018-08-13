# Nginx配置

虽然可以不用代理直接使用Centrifugo ，但从可用性的角度考虑，建议使用代理服务器，这里介绍[Nginx](http://nginx.org/)配置来结合Centrifugo.

Nginx最小版本要求是 – **1.3.13**，因为它是首个能支持Websocket代理的. Centrifugo可以运行在自己的域名或是子目录下(比如 `/centrifugo`).

### Centrifugo运行在不同域名下

```
upstream centrifugo {
    # Enumerate all upstream servers here
    #sticky;
    ip_hash;
    server 127.0.0.1:8000;
    #server 127.0.0.1:8001;
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

#server {
#	listen 80;
#	server_name centrifugo.example.com;
#	rewrite ^(.*) https://$server_name$1 permanent;
#}

server {

    server_name centrifugo.example.com;

    listen 80;

    #listen 443;
    #ssl on;
    #ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    #ssl_ciphers AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
    #ssl_certificate /etc/nginx/ssl/wildcard.example.com.crt;
    #ssl_certificate_key /etc/nginx/ssl/wildcard.example.com.key;
    #ssl_session_cache shared:SSL:10m;ssl_session_timeout 10m;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    gzip on;
    gzip_min_length 1000;
    gzip_proxied any;

    # Only retry if there was a communication error, not a timeout
    # on the Tornado server (to avoid propagating "queries of death"
    # to all frontends)
    proxy_next_upstream error;

    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_set_header Host $http_host;

    location /connection {
        proxy_pass http://centrifugo;
        proxy_buffering off;
        keepalive_timeout 65;
        proxy_read_timeout 60s;
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }
    
    location / {
        proxy_pass http://centrifugo;
    }

    error_page   500 502 503 504  /50x.html;

    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```

支持管理界面的`/socket` 连接:

```
    location /socket {
        proxy_pass http://centrifugo;
        proxy_buffering off;
        keepalive_timeout 65;
        proxy_read_timeout 60s;
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }
```

### 运行在子目录下

```
upstream centrifugo {
    # Enumerate all the Tornado servers here
    #sticky;
    ip_hash;
    server 127.0.0.1:8000;
    #server 127.0.0.1:8001;
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {

    # ... your web site Nginx config

    location /centrifugo/ {
        rewrite ^/centrifugo/(.*)        /$1 break;
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_pass http://centrifugo;
    }

    location /centrifugo/socket {
        rewrite ^/centrifugo(.*)        $1 break;

        proxy_next_upstream error;
        proxy_buffering off;
        keepalive_timeout 65;
        proxy_pass http://centrifugo;
        proxy_read_timeout 60s;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header Host $http_host;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }

    location /centrifugo/connection {
        rewrite ^/centrifugo(.*)        $1 break;

        proxy_next_upstream error;
        gzip on;
        gzip_min_length 1000;
        gzip_proxied any;
        proxy_buffering off;
        keepalive_timeout 65;
        proxy_pass http://centrifugo;
        proxy_read_timeout 60s;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header Host $http_host;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }

}
```

### sticky

你可能注意到`sticky;` nginx的上行流设置，为保证每次连接到的是同一台Centrifugo，我们使用`ip_hash;`参数，但它并不是最佳的，因为有可能有很多客户端连接是来自于同1个ip的，因此最好的解决方案是[nginx-sticky-module](https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/overview)，通过特殊的cookie来保存客户端的连接追踪.

### worker_connections

另外要根据连接数来优化Nginx的`worker_connections`:

```
events {
    worker_connections 40000;
}
```

### 上行流的长连接

请查看操作系统优化的相关知识.
