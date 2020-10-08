---
title: Elastic Beanstalk "Command 01_migrate failed" Error
date: 2020-09-28
categories:
 - Django
tags:
 - AWS
 - Elastic Beanstalk
 - Server deployment
 - Django
 - PostgreSQL
---

Elastic beanstalk에 배포 되어 있는 Django 서버를 AWS PostgreSQL RDS에 migration 하는 도중, migrate fail 에러를 해결 한 방법을 공유합니다.

<!-- more -->

[Django server deployment - Elastic beanstalk에 migration 진행하기](https://kangraemin.github.io/aws/2020/09/28/elasticbeanstalk-migration/)과정 중에

`ToolError: Command 01_migrate failed`을 해결하는 방법을 공유합니다. 

EB를 위한 ec2 인스턴스 OS가 Amazon Linux 2이고, python 3.7을 쓰는 경우 `django-admin migrate`를 지원하지 않는데, 아직 공식 문서가 업데이트 되지 않아 생긴 오류라고 합니다. 

해결 방법은 ec2 인스턴스에서 python 가상 환경을 실행시키고, 또 manage.py를 통해 직접 migrate를 하는 방법이 있습니다. [출처](https://stackoverflow.com/questions/62457165/deploying-django-to-elastic-beanstalk-migrations-failed)

`django-admin migrate`를 `source /var/app/venv/*/bin/activate && python3 manage.py migrate`로 변경하시면 됩니다. 

즉, 이전 글에서 변경된 점은 아래와 같습니다. 

```
container_commands:
  01_migrate:
    command: "django-admin.py migrate"
    leader_only: true
```

를 

```
container_commands:
  01_migrate:
    command: "source /var/app/venv/*/bin/activate && python3 manage.py migrate"
    leader_only: true
```

로 변경하시면 됩니다.