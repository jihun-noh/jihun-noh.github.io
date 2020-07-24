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
SOCKFILE=/home/mybuddy/gunicorn.sock
USER=mybuddy
NUM_WORKERS=3
DJANGO_WSGI_MODULE=MyBuddy.wsgi
LOG_FILE=/logs/mybuddy/`date "+%Y-%m-%d"`.logs

echo "Starting $NAME"

exec /home/mybuddy/.local/bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
  --name $NAME \
  --workers $NUM_WORKERS \
  --user=$USER \
  --bind=unix:$SOCKFILE \
  --log-level=debug \
  --log-file=$LOG_FILE
