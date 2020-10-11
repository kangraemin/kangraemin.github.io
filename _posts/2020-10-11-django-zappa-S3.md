---
title: Zappa를 활용하여 Django 배포하기 - S3에 연결하기
date: 2020-10-11
categories:
 - Django
tags:
 - AWS
 - Zappa
 - Server deployment
 - Django
 - API Gateway
 - Lambda
 - S3
---

Zappa를 활용하여 Django 프로젝트를 API Gateway + Lambda를 활용한 서버리스 아키텍쳐로 배포하는 과정에 대해 공유합니다.    

이번 글에서는 zappa를 활용하여 배포한 Django 앱의 static file 저장소로 AWS S3를 사용하는 방법을 공유합니다. 

<!-- more -->

## Zappa 환경에서 S3 연결하기

[Zappa를 활용하여 Django 배포하기](https://kangraemin.github.io/django/2020/10/03/django-zappa-serverless/) 포스팅에서, zappa init을 할 때 사용 할 S3 버킷 이름을 적어 주었습니다. 

실제로 S3 콘솔에 가보면, 위에서 적은 이름으로 된 S3 버킷이 생성 되어 있는 것을 확인 할 수 있습니다. 

따라서 위에서 생성한 S3 버킷에 Django 앱을 연결하여 `collectstatic`를 진행 해 주려고 했으나, 해당 버킷에 접근 할 수 없다는 아래와 같은 에러만 출력 되었습니다. 

```bash
botocore.exceptions.ClientError: An error occurred (AccessDenied) when calling the PutObject operation: Access Denied
```

이 문제를 해결 하기 위해 IAM 옵션과 bucket의 permission 정말 많이 수정 해 보았지만 위 문제는 계속되었습니다. 

해결방법은, zappa 환경에서 사용하는 S3 버킷에 연결하는 것이 아니라, **새로운 S3 버킷을** 생성하여 사용 하면 문제 없이 `collectstatic`이 작동 하는것을 확인 할 수 있습니다. 

> S3 버킷 생성 및 연결, 이에 필요한 패키지를 설치 하는 부분은 [지난 포스팅](https://kangraemin.github.io/django/2020/09/29/elasticbeanstalk-s3/)을 참고하여 똑같이 진행 해주시면 됩니다.    
> ( S3 bucket 생성, S3와 Django 연결 섹션을 참고하시면 됩니다. )

그리고 `zappa update`와 `zappa manage 여러분의zappa환경이름 collectstatic`을 진행 하면 static file들을 문제 없이 S3에 보내고, 또 zappa 배포 환경에서 S3에 저장된 static file들을 받아 오는 것을 확인 할 수 있습니다. 