---
layout: post
title: 파이썬 django 초기 설정
category: Python
tags: [pyhthon, django]
---

설정 기본
settings.py 파일의 내용은 크게 다음 부분으로 나눌 수 있다.

데이터베이스 설정
템플릿 설정
지역 시각 및 다국어 설정
애플리케이션의 등록
정적 파일 설정
미디어 파일 설정
데이터베이스 설정
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
프로젝트 기본 설정값은 위와 같이 sqlite를 사용하고 있으며 데이터베이스 연동 설정은 다음 링크의 내용을 참고한다.

Django 데이터베이스 연동 설정

템플릿 설정
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
프로젝트 기본 설정값은 위와 같으며 templates 디렉토리 이름을 변경하고자 할 경우 DIRS 변수를 수정한다.

        'DIRS': [os.path.join(BASE_DIR, 'templates')],
지역시각 및 다국어 설정
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_L10N = True
USE_TZ = True
프로젝트 기본 설정값은 위와 같으며 아래와 같이 수정한다.

LANGUAGE_CODE = 'ko-kr'
TIME_ZONE = 'Asia/Seoul'
USE_I18N = True
USE_L10N = True
USE_TZ = True
정적 파일 설정
STATIC_URL = '/static/'
프로젝트 기본 설정값은 위와 같으며 STATIC_URL 변수를 추가한다.

STATIC_ROOT = os.path.join(BASE_DIR, 'static')
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]
미디어 파일 설정
프로젝트 기본 설정값은 없으며 파일 업로드 기능 구현을 위해 다음 설정을 추가한다.

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
runserver 테스트 서버 구동 시에 /media 경로를 올바로 찾지 못하는 문제가 발생한다. 따라서 URL 패턴 매칭을 해주는 다음 코드가 필요하다.

from django.conf.urls.static import static

urlpatterns = [
    ....
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
애플리케이션 등록
bookmark 앱을 생성한 경우 해당 앱 디렉토리 안에 기본 생성 파일은 다음과 같다.

bookmark/
    migrations/
    __init__.py
    admin.py
    apps.py
    models.py
    tests.py
    views.py
Django 버전 1.9부터 애플리케이션의 설정 클래스를 정의하는 apps.py 파일이 추가되었다.

애플리케이션의 등록은 다음과 같이 할 수 있다.

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'bookmark.apps.BookmarkConfig',
]
단순히 bookmark 모듈 이름으로 등록할 수도 있지만 bookmark.apps.BookmarkConfig 설정 클래스 이름으로 등록하는 것이 보다 정확한 방법이다.

설정 실무
settings 패키지로 구현
Django 기본 설정 옵션은 settings.py 모듈 파일 하나로 설정한다. 그러나 실무적으로는 패키지로 만들어 설정을 나눠서 관리한다.

즉, settings.py 파일 하나가 아니라 settings 패키지(디렉토리)를 만들고 이 안에 여러 가지 모듈(파일)을 생성한다.

프로젝트_이름/
    repo/
        app1/
        app2/
        conf/
            __init__.py
            settings/
                base.py
                local.py
                production.py
                test.py
            urls.py
            wsgi.py
        manage.py            
        README.md
        requirements.txt
        .gitignore
    venv/
    run/
        uwsgi.ini
        uwsgi.sock
        gunicorn.sock
    logs/
    ssl/
설정 데이터의 분류
settings 패키지(디렉토리) 안에는 base.py, local.py, production.py 등 여러 개의 모듈(파일)이 들어있다. 이 파일들의 용도와 구분은 다음과 같다.

환경마다 다른 공개적인 설정
local.py, test.py, production.py 등으로 파일을 나누고 저장소에서 관리한다.
모든 환경에서 동일한 공개적인 설정
base.py 파일을 만들고 이를 구체적인 local.py, test,py, production.py 등에서 임포트한다.
환경마다 다른 비공개적인 설정
local.py, test.py, production.py 등으로 파일을 나누고 설정값은 환경변수로 로드한다.
모든 환경에서 동일한 비공개적인 설정
base.py 파일을 만들고 설정값은 환경변수에서 로드하며 구체적인 local.py, test.py, production.py 등에서 임포트한다.
여기서 중요한 것은 공개적인 설정은 저장소에서 코드로 저장되어 관리되지만 비공개 설정은 저장소에 절대 커밋되어서는 안 된다.

비공개 설정 정보하는 방법은 환경변수를 이용하는 방법이 가장 흔하며 시스템에 local.sh, production.sh와 같은 쉘스크립트 파일을 작성하는 것이다.

주요 디렉토리 및 파일 설명
프로젝트 단위로 하위 디렉토리를 가진다.

repo: Django 애플리케이션 소스 코드 디렉토리 = 저장소 커밋 관리
venv: 가상환경 디렉토리
run: 배포 후 소켓 파일 및 연동 설정 파일이 위치하는 디렉토리
logs: 로그 디렉토리
ssl: SSL 키 파일 디렉토리
mkdir 명령어로 repo 디렉토리를 생성 후 디렉토리 안에서 startproject 옵션으로 프로젝트를 만든다.

일반적으로 conf 이름 대신에 프로젝트 이름을 사용하지만 실제로는 프로젝트의 설정 파일이 들어가므로 conf 이름으로 사용한다. 타 프로젝트와 구별하기 위해 최상위 디렉토리 이름을 프로젝트 이름으로 한다.

venv 디렉토리는 종속성 분리의 원칙에 따라 프로젝트의 독립적인 가상환경을 제공한다. 파이썬 3.4 버전부터는 별도의 virtualenv 패키지 설치하지 않고 가상환경 디렉토리를 만들 수 있다.

run 디렉토리는 WSGI 서버가 동작할 때 소켓 파일을 생성하는 디렉토리이다. 보통은 그룹의 소유권을 www-data로 변경한다. 만약 개발 환경에서 manage.py로 테스트 서버를 구동한다면 불필요한 디렉토리이다.

Two scoops of Django 책에서는 디렉토리의 종류를 아래와 같이 구분한다.

저장소 루트
Django 프로젝트 루트 : repo
설정 루트 : conf
