---
title: Django server deployment - Elastic Beanstalk s3 활용하기 ( static file )
date: 2020-09-29
categories:
 - AWS
tags:
 - AWS
 - Elastic Beanstalk
 - Server deployment
 - Django
 - S3
 - Django-storages
---

Django-storages와 boto3를 활용하여 Elastic Beanstalk에서 사용할 static 파일을 s3에 업로드 하는 방법을 공유합니다. 

<!-- more -->

## S3 bucket 생성

Django 앱을 운영하는데에는 static file들이 필요한 경우가 대부분입니다. 

이러한 static file들을 서버와 분리 해놓지 않으면, gitIgnore 되어 있는 static file들은 배포 시 마다 삭제 (초기화)될 것이고, 용량이 매우 커짐에 따라 deploy의 부담이 매우 커질 것 입니다. 

따라서 이런 static file들을 s3에 보관하고자 하고, 필요 할 때 마다 가져오고자 합니다. 

![pic1.png](/assets/images/posts/2020-09-29-elasticbeanstalk-s3/pic1.png)

aws bucket을 생성합니다.

![pic2.png](/assets/images/posts/2020-09-29-elasticbeanstalk-s3/pic2.png)

위 사진 처럼, 아래 두가지에만 체크를 해 줍니다. 

우리는 Django server에서 파일을 업로드 할 수도 있고, 또 Django app에 접속한 사용자가 static file들을 읽을 수 있어야 합니다.

파일 업로드 / 읽기를 위해 django 에서 acl을 통해 파일들을 public으로 지정 할 수 있어야 하기 때문에 위 2가지 경우는 퍼블릭으로 허용 되어야 합니다.

계속 다음을 눌러주시고, s3 bucket 생성을 마무리 해 줍니다. 

## S3와 Django 연결

생성 된 s3 bucket과 django를 연결 하고, 또 로컬에 있는 static file들을 s3에 업로드 해 주여야 합니다. 

s3와 django를 연결하기 위해서는 `boto3`와 `django-storages`패키지가 필요합니다. 

[boto3](https://aws.amazon.com/ko/sdk-for-python/)
[django-storages](https://django-storages.readthedocs.io/en/latest/)

```bash
pipenv install boto3
pipenv install django-storages
```

> 이 두 패키지를 EB에도 설치하기 위해 requirements.txt에도 추가하는것을 잊지 말아 주세요. 

설치 한 뒤, [upload to s3 using django-storages](https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html) 문서를 확인하여 settings.py에 필요한 변수들을 설정 해 줍니다. 

저 같은 경우는 아래의 변수들이 필요 하여 설정 해 주었습니다.

> 여기서 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`는 [이전의 글](https://kangraemin.github.io/aws/2020/09/23/elasticbeanstalk/) 과정 중에 `AWS IAM`생성 과정에서 생성 한 유저에 대한 정보입니다.   
> settings.py에 바로 입력하지 마시고 반드시 환경 변수를 활용하여 입력 해 주세요.
> AWS_DEFAULT_ACL, AWS_S3_OBJECT_PARAMETERS은 [공식 문서](https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html)를 반드시 참고하여 서비스에 필요한 값을 추가 해 주세요.

```
DEFAULT_FILE_STORAGE # 기본 파일 위치 설정
STATICFILES_STORAGE # 스태틱 파일 위치 설정
AWS_ACCESS_KEY_ID # AWS IAM ID
AWS_SECRET_ACCESS_KEY # AWS IAM PASSWORD
AWS_STORAGE_BUCKET_NAME # 생성한 s3 BUCKET 이름 
AWS_DEFAULT_ACL # 저같은 경우는 public-read로 지정 해 주었습니다. 공식문서를 반드시 참조 해주세요.
AWS_S3_OBJECT_PARAMETERS # 저는 캐싱을 사용하기 위해 {"CacheControl": "max-age=86400"}값을 사용하였습니다. 공식문서를 반드시 참조 해주세요.
AWS_S3_CUSTOM_DOMAIN # 공식 문서를 참조해주세요. ( cdn 사용이냐, s3 사용이냐에 갈리지만, 여기선 s3 이므로 저는 f"{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com"를 사용 하였습니다.)
STATIC_URL # django 프로젝트에서 사용할 static 파일을 경로를 지정 해주세요. 저같은 경우는 f"https://{AWS_S3_CUSTOM_DOMAIN}/static/"을 사용 하였습니다.
```

## EB 명령어에 collect static 추가

collect static 명령어를 EB에 추가해서 EB에 흩뿌려진 static 파일들을 STATIC_ROOT로 모아줍니다.

[앞의 글](https://kangraemin.github.io/aws/2020/09/28/elasticbeanstalk-migration/)에서 처럼, `.ebextension`안의 `django.config`속에 아래와 같은 코드를 추가 해 줍니다. ( 반드시 `container_commands`의 하위에 넣어야 합니다. )

```
container_commands:
  02_collectstatic:
    command: "source /var/app/venv/*/bin/activate && python3 manage.py collectstatic --noinput" # no input -> 물어보는 값 모두 yes
```

커밋 후 eb deploy 진행 이후 s3를 확인하시면, 아래와 같이 static file들이 업로드 된 것을 확인 할 수 있습니다. 

![pic3.png](/assets/images/posts/2020-09-29-elasticbeanstalk-s3/pic3.png)

![pic4.png](/assets/images/posts/2020-09-29-elasticbeanstalk-s3/pic4.png)

> 반드시 eb open을 통해 eb 환경에 들어가서, 제대로 static 파일들이 불려 져 오는것을 확인 해주세요.