---
title: Django 서버 에러 받기 - sentry 사용하기
date: 2020-09-28
categories:
 - AWS
tags:
 - AWS
 - Elastic Beanstalk
 - Server deployment
 - Django
 - Sentry.io
---

Elastic beanstalk에 배포 되어 있는 Django 서버에서, 서버 에러가 날 시 에러 리포트를 받아 볼 수 있는 Sentry.io를 적용하는 방법을 공유합니다.

<!-- more -->

## Sentry.io

Django 서버가 DEBUG모드 일 때엔, 서버에서 작업 도중 에러가 나면 아래와 같은 에러 뷰를 볼 수 있었습니다. 

![pic1.png](/assets/images/posts/2020-09-28-django-sentry/pic1.png)

하지만, EB에 배포된 경우 보안상의 이유로 DEBUG는 True로 설정 할 수 없어 위와 같은 에러뷰를 확인 할 수 없습니다. 

따라서 에러가 났을때 그에 대응하기 힘들어집니다. 

[sentry.io](https://sentry.io/welcome/)를 활용 하면 원격 서버에서 에러가 났을 시, sentry.io 웹에서 해당 에러에 대한 자세한 내용을 확인 할 수 있습니다.

회원가입 및 로그인을 하시고, create new project를 진행합니다.

![pic2.png](/assets/images/posts/2020-09-28-django-sentry/pic2.png)

그리고 EB를 통해 배포 되어 있다면, requirements.txt를 통해 `sentry-sdk`를 설치 해 주시고, 로컬 환경이라면 `pip install sentry-sdk`를 이용하여 설치 해 주세요. ( 가상환경이라면 `pipenv` )

```python
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn="개인 url이 sentry 화면에 나와 있습니다.",
    integrations=[DjangoIntegration()],
    traces_sample_rate=1.0,

    # If you wish to associate users to errors (assuming you are using
    # django.contrib.auth) you may enable sending PII data.
    send_default_pii=True
)
```

그리고 django의 settings.py에 위와 같은 코드를 추가 해 줍니다. ( dsn에 유의 해주세요 )

```python
from django.urls import path

def trigger_error(request):
    division_by_zero = 1 / 0

urlpatterns = [
    path('sentry-debug/', trigger_error),
    # ...
]
```

위의 코드를 추가하여 test url을 추가 할 수 있습니다. 

![pic3.png](/assets/images/posts/2020-09-28-django-sentry/pic3.png)

DEBUG가 True가 아니더라도, 서버에서 에러가 발생하면 위와 같이 에러 알림이 오고 이를 클릭하면 

![pic4.png](/assets/images/posts/2020-09-28-django-sentry/pic4.png)

위와 같이 에러에 대한 자세한 정보를 확인 할 수 있습니다. 

Sentry.io는 무료로 일정 부분 사용 할 수 있기 때문에 혼자 개발 시 충분히 사용 할 만 한것 같습니다. 