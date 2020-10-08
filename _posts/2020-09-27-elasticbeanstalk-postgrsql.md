---
title: Elastic Beanstalk - Django AWS PostgreSQL RDS 연결하기
date: 2020-09-27
categories:
 - Django
tags:
 - AWS
 - Elastic Beanstalk
 - Server deployment
 - Django
 - PostgreSQL
---

AWS PostgreSQL RDS를 Django환경인 Elastic beanstalk 개발환경에 연결 및 셋팅하는 과정을 공유합니다. 

<!-- more -->

## PostgreSQL 설치 및 python package 설치 

따로 DB를 설정 해 주지 않은 경우, Django는 SQLite를 기본 DB로 사용합니다. 

이 때, sqlite는 EB환경과 종속적인 관계가 되며, EB환경이 초기화 되거나, 문제가 생기면 DB도 같이 사용하지 못하는 경우가 발생합니다. 

이 경우를 방지하기 위해, DB와 EB를 분리 해 줄 필요가 있으며, 이를 위해 AWS PostgreSQL RDS을 사용하고자 합니다. 

이때, postgresql을 EB( OS : Amazon Linux 2 )에 설치 한 뒤, postgresql을 python과 연결 해주어야 합니다. 

### eb config 활용하여 postgresql 설치 

EB에 `yum install postgresql-devel` 명령어를 동작 시키면, postgresql을 EB에 설치 할 수 있습니다. 

일일이 입력 해주지 않아도, EB의 `.ebextensions`를 통해 EB 배포 시 진행 할 명령어들을 미리 설정 할 수 있어 이를 활용하고자 합니다 ( 참고 : [AWS 공식 문서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/customize-containers-ec2.html) )

방법은 아래와 같습니다. 

1. `.ebextensions` 폴더에 `packages.config` ( 이 파일의 네이밍은 공식적으로 지정 된 것은 아니며, package의 config 라는 뜻으로 일반적으로 많이 사용 되는듯 합니다. ) 파일을 생성

2. `packages.config`파일 안에 아래의 명령어를 입력

```bash
packages:
  yum: # yum 패키지 매니저를 활용하여 
    postgresql-devel: [] # postgresql-devel을 설치한다. 
```

위의 코드는 EB환경에 `pipenv install psycopg2`와 같은 명령어를 실행 할 수 있음을 확인 할 수 있습니다. 

### Python와 postgresql을 연결하기 위한 psycopg2 설치

Python과 PostgreSQL을 연결 하기 위해 [psycopg2](https://pypi.org/project/psycopg2/)를 사용 합니다. 

Local에서 다운 받는다면, `pipenv install psycopg2` 명령어로 진행하시면 되지만, 우린 EB 환경에서 이 패키지를 하기 위해선 EB환경에 해당 패키지를 설치 해 주어야 합니다. 

따라서 [앞의 글](https://kangraemin.github.io/aws/2020/09/23/elasticbeanstalk/)에서 처럼, requirement.txt에 `psycopg2==2.8.4`를 추가하여 커밋 후 (git 연동을 진행 하셨다면 )eb deploy를 진행 해 줍니다. 

```bash
Error: pg_config executable not found.

pg_config is required to build psycopg2 from source. Please add the directory

containing pg_config to the $PATH or specify the full executable path with the

option:
```

저는 psycopg2를 설치하려고 하면, 위의 에러가 자꾸 나 psycopg2-binary를 설치하니 제대로 설치가 되더라구요. 혹시 psycopg2를 설치하는데 자꾸 에러가 나시면 binary버전을 설치 해 보세요. ( 출처 : [https://stackoverflow.com/questions/60005953/aws-beanstalk-cant-install-psycopg2](https://stackoverflow.com/questions/60005953/aws-beanstalk-cant-install-psycopg2))

## RDS 생성 및 연결

[공식문서](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html)를 참고하여 RDS 인스턴스를 생성 해줍니다.

이 때, 저는 postgresql 9.6, free tier로 생성 해 주었습니다. 

생성 시 적어 두었던 마스터 사용자 이름과, 비밀번호를 기억 해 둔 뒤 생성을 완료합니다. 

django settings.py에, Database를 다음과 같이 설정 해 줍니다. 

```python
if DEBUG:
    DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.sqlite3",
            "NAME": os.path.join(BASE_DIR, "db.sqlite3"),
        }
    }
else:
    DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.postgresql",
            "HOST": os.environ.get("RDS_HOST"),
            "NAME": os.environ.get("RDS_NAME"),
            "USER": os.environ.get("RDS_USER"),
            "PASSWORD": os.environ.get("RDS_PASSWORD"),
            "PORT": "5432",
        }
    }
```

Elastic beanstalk에 환경 변수 설정을 하는 방법은, [AWS 공식 문서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/environment-configuration-methods-after.html)를 참고 해주세요. 

참고로 저는 아래의 사진처럼 Elastic Beanstalk 콘솔 - 구성 부분을 활용하여 software의 변수들을 설정 해 주었습니다. 

![pic1.png](/assets/images/posts/2020-09-27-elasticbeanstalk-postgrsql/pic1.png)

이후 commit & eb deploy 를 진행한 뒤 환경으로 이동 하면, 무한 로딩을 확인 하실 수 있습니다. 

이는 RDS의 security group이 설정 되지 않았기 때문입니다. ( EB와 RDS는 독립적인 환경 )

따라서 RDS와 EB가 서로 소통 할 수 있도록 보안 그룹을 설정 해 주어야 합니다. 

## RDS Security group

RDS 콘솔에 들어가서, 생성 한 RDS 클릭 후 `보안 그룹 규칙` 탭을 확인합니다. 

![pic2.png](/assets/images/posts/2020-09-27-elasticbeanstalk-postgrsql/pic2.png)

위 사진에 나온 `보안 그룹 규칙`에서, `EC2 Security Group - Inbound`유형의 보안 그룹 규칙에 접속한 뒤 `인바운드 규칙` 클릭 후 `인바운드 규칙 편집`을 클릭합니다.  

![pic3.png](/assets/images/posts/2020-09-27-elasticbeanstalk-postgrsql/pic3.png)

위 사진에 나온 것 처럼, Elastic beanstalk 환경들이 확인 되는데 연결 하고자 하는 EB환경을 선택 해 준 뒤, 규칙 저장을 눌러줍니다.

위 과정을 풀어서 설명하면, Elastic beanstalk에는 많은 AWS 서비스들 ( S3, Autoscaling, EC2 ... )등이 서로 통신을 하기 위해 설정된 보안 그룹이 있습니다.

이 보안 그룹 속에서는 서로 통신하는것이 자유롭도록 설정 되어 있습니다. 

하지만 좀전에 생성한 RDS는 이 그룹에 속해있지 않고, inbound 규칙 ( RDS에 접속 하는 데에 대한 규칙 )에도 아무것도 접속하지 못하도록 되어 있습니다. (outbound, 외부 통신은 아무데나와 할 수 있도록 설정 되어 있습니다. )

따라서 RDS를 EB의 보안그룹에 속하도록 ( EB와 소통 할 수 있도록 )설정 해 준것입니다. 

다시 Elastic beanstalk 배포 환경으로 접속 해 보면, 로컬에서 이미 개발한 환경을 deploy 하는 과정 중이라면 데이터베이스 migration이 되지 않았기 떄문에 배포 환경의 setting.py의 DEBUG가 True라면 django error template이 보일 것이고, 아니라면 서버 에러가 뜰 것입니다.

그렇다면 RDS와 EB의 연동은 잘 되었고, 이제 남은 일은 Django 앱 Database를 RDS에 migration 해주는 일입니다. 해당 작업은 다음 포스팅때 진행 하도록 하겠습니다. 