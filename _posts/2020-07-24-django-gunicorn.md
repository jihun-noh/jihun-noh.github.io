---
layout: post
title: gunicorn 설치
category: Python
tags: [pyhthon, django, gunicorn]
---
-> pip install gunicorn  
-> 테스트  
-> gunicorn test.wsgi:application --bind 0.0.0.0:8000  
-> 실행파일  
-> #!/bin/bash

NAME=MyBuddy  
USER=mybuddy  
NUM_WORKERS=3  
DJANGO_WSGI_MODULE=MyBuddy.wsgi  
LOG_FILE=/logs/mybuddy/`date "+%Y-%m-%d"`.log  

echo "Starting $NAME"  

exec gunicorn ${DJANGO_WSGI_MODULE}:application \  
  --bind=0.0.0.0:8000 \  
  --name $NAME \  
  --workers $NUM_WORKERS \  
  --user=$USER \  
  --log-level=debug \  
  --log-file=$LOG_FILE  
  
-> systemctl 등록  
-> /etc/systemd/system/gunicorn.service 생성
[Unit]  
Description=gunicorn daemon  
After=network.target  
  
[Service]  
User=mybuddy  
WorkingDirectory=/project/mybuddy/mybuddyvenv/bin/myBuddy/  
EnvironmentFile=/project/mybuddy/mybuddyvenv/bin/myBuddy/.env  
ExecStart=/home/mybuddy/.local/bin/gunicorn \  
        --workers 3 \  
        --bind 0.0.0.0:8000 \  
        MyBuddy.wsgi:application  
  
[Install]  
WantedBy=multi-user.target  
  
-> systemctl daemon-reload  
-> systemctl start gunicorn  
