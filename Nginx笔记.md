```shell
user root;

worker_processes  1;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;
        location / {
            root   /home/ftpuser/www;
            index  index.html index.htm;
        }
        
    }

 server {
        listen 8080;
        server_name www.huang.com;
        location / {
                 proxy_pass http://tomcat_server;
                 index index.jsp index.html index.htm;
        }
}

upstream tomcat_server{
        server 192.168.0.102:80 weight=10;
        server 192.168.0.103:80 weight=5;
        }

    }
```

