---
layout: post
title: nginx 설치
category: Python
tags: [pyhthon, django, nginx]
---
-> yum install nginx  
-> 설정  
-> /etc/nginx/conf.d  
->  server {  
        listen       80;  
        server_name  ip;  
        location / {  
                proxy_pass http://127.0.0.1:8000;  
        }  
        location /static {  
                autoindex on;  
                alias /project/mybuddy/mybuddyvenv/bin/myBuddy/staticfiles/;  
        }  
        error_page 404 /404.html;  
            location = /40x.html {  
        }  
  
        error_page 500 502 503 504 /50x.html;  
            location = /50x.html {  
        }  
}    
-> systemctl restart nginx
