---
title: Zappa를 활용하여 Django 배포하기 - MySQL RDS에 연결 & Migrate 하기
date: 2020-10-10
categories:
 - Django
tags:
 - AWS
 - Zappa
 - Server deployment
 - Django
 - API Gateway
 - Lambda
 - RDS
 - MySQL
---

Zappa를 활용하여 Django 프로젝트를 API Gateway + Lambda를 활용한 서버리스 아키텍쳐로 배포하는 과정에 대해 공유합니다.    

이번 글에서는 zappa를 활용하여 배포한 Django 앱의 Databases로 AWS MySQL RDS를 사용하는 방법을 공유합니다. 

<!-- more -->

## RDS 생성

[공식문서](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MySQL.html)를 참고하여 RDS 인스턴스를 생성 해줍니다.

이 때, 저는 MySQL RDS 간편 생성, Free tier로 진행 하였습니다.

## 패키지 설치 및 RDS와 연결하기

### 1. Package 설치 

`pip install mysqlclient mysql-connector-python`로 로컬에 MySQL과 django 앱을 연결해주기 위한 패키지들을 설치 해 줍니다.

### 2. MySQL RDS생성 및 연결 

Django 앱과 RDS연결에 필요한 정보들인 Endpoint, Port, User name, User password 등을 `.env`나 환경 설정 파일에 적어 둡니다.

```python
DATABASES = {
    "default": {
        "ENGINE": os.environ.get("DB_ENGINE"),
        "HOST": os.environ.get("DB_HOST"),
        "NAME": os.environ.get("DB_NAME"),
        "USER": os.environ.get("DB_USER"),
        "PASSWORD": os.environ.get("DB_PASSWORD"),
        "PORT": os.environ.get("DB_PORT")
    }
}
```

적어둔 값을 위와 같이 `settings.py`에 추가 해 줍니다. 

여기서 DB ENGINE의 값을 `django.db.backends.mysql`를 사용하여 연결을 시도하면

```text
NameError: name '_mysql' is not defined
```

위의 에러가 뜹니다. 이를 해결하기 위해서는 DB ENGINE의 값을 `mysql.connector.django`로 사용해 주어야 합니다. [출처 : MySQL 공식 문서](https://dev.mysql.com/doc/connector-python/en/connector-python-django-backend.html)

## 보안 설정 및 migration

### 1. 보안 설정

security group 설정의 inbound 설정에서 source 항목을 default로 설정 해 주세요.

[지난번 글](https://kangraemin.github.io/django/2020/09/27/elasticbeanstalk-postgrsql/)에서는 security group에 EB환경에 사용하던 인스턴스를 추가 해 주어 접근을 허용 해 준 것이라면, 이번엔 zappa를 활용해 생성한 람다 환경을 security 그룹에 속하도록 추가 해 주어야 합니다.

이를 위해 aws의 lambda 콘솔에 가셔셔, zappa에서 생성한 람다 환경으로 들어 가 주시고 VPC 항목을 설정 해 주어야 합니다. 

![pic1.png](/assets/images/posts/2020-10-10-django-zappa-RDS/pic1.png)

Edit VPC를 누르시고

![pic2.png](/assets/images/posts/2020-10-10-django-zappa-RDS/pic2.png)

생성한 RDS의 VPC와, subnet, 그리고 security group항목에 rds가 속해있는 security group을 추가 해 줍니다. 

### 2. Create db & Migration

[지난번 글](https://kangraemin.github.io/django/2020/09/28/elasticbeanstalk-migration/) EB CLI를 활용하여 Django를 PostgreSQL RDS에 연결 했을 땐, RDS에 아무런 DB가 생성 되어 있지 않아도 Migration 도중 DB를 생성 할 수 있었습니다. 

하지만 zappa를 활용하여 MySQL RDS에 migration을 할 때는 db를 자동으로 생성 해 주지 못해 아래의 코드를 추가 해 custom command를 추가 해 줍니다. 

#### create_db command 추가하기 

```python
import sys
import logging
import mysql.connector
import os

from django.core.management.base import BaseCommand, CommandError
from django.conf import settings

rds_host = os.environ.get("DB_HOST")
db_name =  os.environ.get("DB_NAME")
user_name =  os.environ.get("DB_USER")
password =  os.environ.get("DB_PASSWORD")
port = os.environ.get("DB_PORT")

logger = logging.getLogger()
logger.setLevel(logging.INFO)

class Command(BaseCommand):
    help = 'Creates the initial database'

    def handle(self, *args, **options):
        print('Starting db creation')
        try:
            db = mysql.connector.connect(host=rds_host, user=user_name,
                                 password=password, db="mysql", connect_timeout=5)
            c = db.cursor()
            print("connected to db server")
            c.execute("""CREATE DATABASE 생성할DB이름;""")
            c.execute("""GRANT ALL ON 생성할DB이름.* TO '사용할관리자ID'@'%'""")
            c.close()
            print("closed db connection")
        except mysql.connector.Error as err:
            logger.error("Something went wrong: {}".format(err))
            sys.exit()
            
```

#### Migration

위 작업을 진행 한 뒤, 로컬 환경에서 `python manage.py makemigrations`명령어를 통해 migration할 파일을 생성 해 줍니다. 

create_db command를 추가 하고, makemigrations작업 까지 완료 되었다면 `zappa update`를 통해 배포된 zappa환경 update를 진행합니다. 

완료되면, `zappa manage 여러분의zappa환경이름 create_db`를 통해 db를 생성 해 줍니다.

db 생성이 완료 된 뒤 `zappa manage 여러분의zappa환경이름 migrate`를 통해 migration을 진행 해 줍니다. 

> 위 과정 중 틀린 내용이 있어 코멘트 주시면 반드시 수정 하겠습니다    
> 많은 지적 부탁드리겠습니다     
