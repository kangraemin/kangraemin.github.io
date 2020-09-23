---
title: Android Cook Book (2) - Git / Github / SourceTree를 활용하여 안드로이드 코드 관리하기
date: 2020-07-25
categories:
 - aws
tags:
 - aws
 - Elasticbeanstalk
 - Server deployment
---

Django 서비스 배포를 위해 AWS의 elasticbeanstalk에 대해 공부한 내용을 정리합니다. 

<!-- more -->

## Elasticbeanstalk

- 자동적으로 ec2 인스턴스를 생성합니다.
- ec2 생성 및 배포를 위한 단축키로 생각하면 좋습니다.
- git branch와 연동하여 test / service 서버 배포를 쉽게 할 수 있습니다.
- 로드밸런서, 오토 스케일링 등 배포에 필요한 작업들을 자동적으로 진행 해 줍니다. 

### EB CLI를 활용하여 Django 앱 배포 
참고 : [https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/create-deploy-python-django.html](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/create-deploy-python-django.html)

```bash
pipenv install awsebcli --dev
eb init 
```

- AWS IAM을 통해 사용자를 추가하고, access-id / secret-key를 발급 받아 EB init 도중에 사용자를 입력하여 줍니다. 
- 이때 user의 secret-key는 두번 다시 확인 할 수 없으므로, 다운로드 받아 잘 보관 해 두는것이 좋습니다. 
- 실제 서버에서는 python manage.py runserver가 아닌, WSGI를 활용하여 서버를 실행합니다. ( 속도 등의 다양한 이유로 ) 
- 프로젝트의 root에 `.ebextensions`폴더를 생성 하고, 해당 폴더 속에 `django.config`파일을 생성 한 뒤 아래의 내용을 입력합니다. 
```
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: config.wsgi 
```
- config/wsgi 가 아닌 것에 주의 ! / 를 사용하면 제대로 인식하지 못함 ( 플랫폼 정보 : arn:aws:elasticbeanstalk:ap-northeast-2::platform/Python 3.7 running on 64bit Amazon Linux 2/3.1.1 )

- 아래의 명령어를 통해 EB환경의 로그 ( django error log / http connection log / eb log ... ) 들을 확인 할 수 있습니다. 
```bash
eb logs 
```

- Git / EB 연동 하기 
참고 : [https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/eb3-cli-git.html](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/eb3-cli-git.html)

- 가상 환경에 설치 된 패키지들을 EB에 설치 해 주려면, 프로젝트의 root에 requirements.txt를 만들어 놓으면 EB가 이를 인식하여 패키지들을 자동적으로 설치 해 줍니다.

- `pipenv_to_requirements` 패키지를 활용하면 requirements.txt를 쉽게 생성 할 수 있습니다.

- Git과 EB가 연동 되어 있다면, 커밋 된 내용만 EB에 반영되므로 배포 전 반드시 커밋을 하셔야 합니다. 

```bash
eb deploy # EB 배포
eb open # EB 확인 
```

### Elasticbeanstalk 장단점
장점 
- 코드 배포가 상당히 간단합니다. 
- ec2 인스턴스 생성, 로드밸런서 설정 등을 자동적으로 진행 해 줍니다. 
- git 연동도 상당히 쉬워 서버 배포에 어려움이 없습니다. 

단점
- 로드밸런서, S3등 자동적으로 만들어지는 서비스가 많아 가격이 비쌀 수 있을 것 같습니다. ( 정확히 확인 되지 않음 ) 
- AWS에 이미 구축된 서비스의 경우, EB로 옮기기엔 쉽지 않습니다. 