---
title: Django server deployment - Elastic Beanstalk에 migration 진행하기
date: 2020-09-28
categories:
 - AWS
tags:
 - AWS
 - Elastic Beanstalk
 - Server deployment
 - Django
 - PostgreSQL
---

Elastic beanstalk에 배포 되어 있는 Django 서버를 AWS PostgreSQL RDS에 migration 하는 과정을 공유합니다.

<!-- more -->

## Add Migration command to EB config

로컬에서는 Django model 작업을 진행 한 뒤, manage.py를 통해 makemigration 및 migration을 진행 할 수 있습니다. 

이 작업을 [이전 포스팅](https://kangraemin.github.io/aws/2020/09/27/elasticbeanstalk-postgrsql/)에서 yum을 통해 `postgresql-devel`을 설치 한 것 처럼, `.ebextension`에 명령어를 추가 해 줌으로써 통해 진행 할 수 있습니다. 

[AWS 공식 문서](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-django.html)의 `Add a database migration configuration file` 부분에서 확인 할 수 있습니다. 

![pic1.png](/assets/images/posts/2020-09-28-elasticbeanstalk-migration/pic1.png)

위 공식문서 처럼 작업하기 위해, [이전 포스팅](https://kangraemin.github.io/aws/2020/09/23/elasticbeanstalk/)에서 만들었던 `.ebextension`안의 `django.config`속에 아래와 같은 코드를 추가 해 줍니다.

```
container_commands:
  01_migrate:
    command: "django-admin.py migrate"
    leader_only: true
option_settings:
  aws:elasticbeanstalk:application:environment:
    DJANGO_SETTINGS_MODULE: config.settings
```

- `container_commands`는 EB에 실행 시킬 command들을 담는 공간입니다. 

- `n(숫자)_내릴명령어`로 command들을 구분 짓습니다. 

- `n(숫자)_내릴명령어`의 알파벳 순서대로 command를 실행 시킵니다. 

- `command`에는 실행 시킬 command를 입력 합니다.

- `leader_only`는 Elastic beanstalk에 aws auto scaling에 따라 여러 인스턴스 환경에서 실행 될 때, leader instance만 해당 커멘드를 실행 할 지 말지 결정짓는 값입니다. 해당 값이 True라면, 여러 인스턴스에 배포할 때 한 번만 실행됩니다.

- `option_settings`는 Elastic beanstalk를 구성하고 있는 auto scaling, loadbalancer, elasticbeanstalk과 같은 구성 요소들의 셋팅 값을 설정하는 공간입니다. 

- `aws:elasticbeanstalk:application:environment`는 elasticbeanstalk의 환경 변수를 설정 하겠다는 의미입니다. 

- `DJANGO_SETTINGS_MODULE: config.settings`는 DJANGO_SETTINGS_MODULE의 값을 config.settings로 설정 하겠다는 의미입니다. 

위 코드를 추가 하고, eb deploy를 진행 한 뒤 배포 환경에 접속 해보면, migration이 된 것을 확인 할 수 있습니다. 

위 과정 중 `ToolError: Command 01_migrate failed`로 실패 한다면, [이 글](https://kangraemin.github.io/aws/2020/09/28/elasticbeanstalk-migration-error/)을 참조 해 주세요