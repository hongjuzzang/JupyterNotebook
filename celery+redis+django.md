# celery + redis + django 설정하기  
```
$ pip install 'celery[redis]'
$ sudo apt-get install tcl8.5
```
make, make test, make install 을 실행  
```
sudo apt-get install redis-server  
sudo systemctl enable redis-server.service
redis-cli ping # 하면 pong으로 오면 성공
```
코드에서 쓰이는 라이브러리 추가 설치  
```
 pip install django-celery-beat
 pip install django-celery-results
```
# backend/backend/settings.py
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django_celery_beat',
    'django_celery_results',
    'scraper',
]
 
 
# Celery
CELERY_BROKER_URL = 'redis://localhost:6379'
CELERY_RESULT_BACKEND = 'redis://localhost:6379'
CELERY_ACCEPT_CONTENT = ['application/json']
CELERY_TAST_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Seoul' #Celery beat가 스케줄러이기 때문에 시간에 대한 정의를 해야함

```
# backend/backend/celery.py 파일 생성
```
from __future__ import absolute_import
 
import os
 
from celery import Celery
 
# Django의 세팅 모듈을 Celery의 기본으로 사용하도록 등록합니다.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'backend.settings') # project name
 
app = Celery('backend') # project name
 
 
# 문자열로 등록한 이유는 Celery Worker가 자식 프로세스에게 configuration object를 직렬화하지 않아도 된다는것 때문
# namespace = 'CELERY'는 모든 celery 관련한 configuration key가 'CELERY_' 로 시작해야함을 의미함
app.config_from_object('django.conf:settings', namespace = 'CELERY')
 
# task 모듈을 모든 등록된 Django App configs에서 load 함
app.autodiscover_tasks()
 
 
@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))

```

# backend/backend/__init__.py 수정
```
from __future__ import absolute_import   # Django가 시작할 때  shared_task가 이 앱을 이용할 수 있도록 app이 항상 import 되게 해준다. 
from .celery import app as celery_app
```

# backend/api/tasks.py 생성, 어떤 함수를 동작시킬것인지
```
from celery.schedules import crontab
 
@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
 
app.conf.beat_schedule = {
    'add-every-minute-contrab': {
        'task': 'multiply_two_numbers',
        'schedule': crontab(),  # 1분마다
        'args': (16, 16),
    },
    'add-every-5-seconds': {
        'task': 'multiply_two_numbers',
        'schedule': 5.0,  # 5초마다
        'args': (16, 16)
    },
    'add-every-30-seconds': {
        'task': 'tasks.add',
        'schedule': 30.0, # 30초마다
        'args': (16, 16)
    },
}

```

$ python manage.py makemigrations
$ python manage.py migrate

$ celery -A [파일이름:backend] worker -l info

$ views.py에서 tasks.py에 있는 함수를 [함수이름].delay()로 실행

출처: https://wangin9.tistory.com/entry/celery [잉구블로그]
[웹서비스 비동기(백그라운드) 프로세스 만들기 (feat. Django, Celery) 4](https://ohohs.tistory.com/entry/%EC%9B%B9%EC%84%9C%EB%B9%84%EC%8A%A4-%EB%B9%84%EB%8F%99%EA%B8%B0%EB%B0%B1%EA%B7%B8%EB%9D%BC%EC%9A%B4%EB%93%9C-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EB%A7%8C%EB%93%A4%EA%B8%B0-feat-Django-Celery-4?category=695701)