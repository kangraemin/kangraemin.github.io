---
title: Zappa를 이용하여 AWS Lambda에 Django 배포하기 (1)
date: 2020-10-03
categories:
 - AWS
tags:
 - AWS
 - Zappa
 - Server deployment
 - Django
 - API Gateway
 - Lambda
---

Zappa를 활용하여 Django 프로젝트를 API Gateway + Lambda를 활용한 서버리스 아키텍쳐로 배포하는 과정에 대해 공유합니다. 

<!-- more -->

## 서버리스

### 서버리스란 ?

서버리스의 단어 그대로를 직역 해보자면, `서버가 없는`으로 해석 될 수 있습니다. 

하지만, 실제 서버가 없이 클라이언트-서버 시스템을 구현 할 수는 없습니다. 

따라서 서버리스도 실제 서버가 존재하지 않는 것은 아닙니다. 

AWS Lambda를 활용한 서버리스 시스템 에서는 백엔드 코드를 Lambda에 올려 놓은 이후에 **요청이 들어 올 때마다** 코드를 가동시킵니다. 

즉, EC2를 대여해 서버 배포를 한 경우와 같이 서버 앱을 **항상 가동**시키는 것이 아니라, **요청이 들어 올 때마다** 코드를 가동시키는 방식입니다. 

서버가 존재하긴 하지만 유저로 부터 요청이 들어 올 때마다 함수(코드)가 돌아가는 시간동안 서버를 **대여했다가 함수 실행이 끝나면 반납 하는 구조** 인것입니다.  

### 서버리즈 장점 

1. 개발의 편리성 

예전에 AWS EC2를 활용한 클라이언트-서버 시스템은 AWS EC2에 본인이 만든 백엔드 코드를 업로드 하고, 앱을 가동시켜 서버 역할을 하게 해주는 컴퓨터를 대여하여 서버를 구축하는 방식이었습니다. 

이 경우엔 유저가 갑자기 늘어나거나, 많은 업무를 순간적으로 처리학기 위해서 AWS의 로드밸런싱, Auto Scaling 서비스들을 사용 해야만 했습니다. ( 저는 해당 사항이 필요 하지 않았습니다. )

[앞의 글](https://kangraemin.github.io/aws/2020/09/23/elasticbeanstalk/)에서 사용했던 Elastic beanstalk은 이러한 Auto scaling, EC2, Load balancing등을 AWS에서 알아서 설정 해주도록 도와주는 서비스 였습니다. 

하지만 Lambda를 활용한 서버리스 시스템은 이미 Lambda안에 Auto scaling이나 로드밸런싱이 **적용 되어 있기 때문**에 개발자가 따로 설정을 해주지 않아도 됩니다.

2. 가격

이 대여 한 만큼에 대한 비용만 지불 하면 되기 때문에, 서버를 계속해서 가동 시키는 형태 보다 저렴 할 수 있습니다. ( 실제 람다의 가격 : 요청 1백만 건당 0.20 USD, AWS 서울 리전 기준 [AWS Lambda pricing](https://aws.amazon.com/ko/lambda/pricing/) )

참고 : [로켓펀치 : 300원에 200만뷰 소화하기 – 서버리스 아키텍처 AWS 람다(Lambda) 활용 사례](https://blog.rocketpunch.com/2017/07/02/2-million-pv-with-300-krw/)

### 서버리스 단점

1. 서버 공급 회사에 강한 의존성을 띄게 됩니다. 

AWS Lambda를 활용한 서버리스 서버를 구축하여 서비스 중인 경우에, AWS회사가 갑자기 망해버리는 경우 ... 도 있을 수 있고, 또 가격 정책을 AWS가 변경 할 경우 대응 하기 힘들어집니다. 

또 한 회사에 많이 의존 하게 되기 떄문에, AWS를 사용하다가 Azure나 다른 서비스로 이관 시키는 작업이 필요 할 경우, 옮겨 가는것이 힘이 듭니다. 

2. 학습이 어려움 

AWS Lambda를 활용한 서버리스 서버 구축의 경우, 한국어 자료가 EC2 배포에 비해 빈약 합니다. 

또 백엔드 프레임워크 ( django, node.js )들을 배포 할 때 참고 할만한 자료도 많지 않고, 해당 프레임워크의 버전 별로 에러 대응 방식이 다른 경우도 있어 학습이 어려운 편입니다. 

## Zappa 

Zappa는 AWS 기능들을 활용한 Serverless 구조를 활용하여 Python 백엔드 코드를 배포하는데 도움을 주는 패키지 입니다. 

Zappa를 통해 API Gateway, S3 연동 등을 좀 더 쉽게 할 수 있으며 배포 과정도 상당히 쉽게 배포 할 수 있도록 도와줍니다. 

[Zappa 공식 Github](https://github.com/Miserlou/Zappa)

1. zappa 설치 

```bash
pip install zappa 
```

> 이 과정 중에, 이미 Elastic beanstalk CLI가 설치 되어 있는 가상환경에 설치하려고 한다면 ebcli와 zappa가 충돌을 일으키게 됩니다. 둘 중 하나를 삭제 후 재설치 하시면 해결됩니다. 

2. AWS IAM 정보 발급 

AWS에 접속 하셔셔, `AWS IAM`계정을 생성 해 주세요. ( [공식 문서](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/introduction.html) 참조 )

그리고 `/.aws/credentials`파일을 생성하고, `/.aws/credentials`파일에 아래와 같이 입력 해 줍니다.

```bash
[default]
AWS_ACCESS_KEY_ID= # IAM에서 생성한 ACCESS_KEY
AWS_SECRET_ACCESS_KEY= # IAM에서 생성한 SECRET_KEY

[zappa]
AWS_ACCESS_KEY_ID= # IAM에서 생성한 ACCESS_KEY
AWS_SECRET_ACCESS_KEY= # IAM에서 생성한 SECRET_KEY
```

이렇게 생성 해놓으면, AWS별로 KEY값들을 관리하기 수월 해 집니다. 

3. Set up zappa

```bash
$zappa init

███████╗ █████╗ ██████╗ ██████╗  █████╗
╚══███╔╝██╔══██╗██╔══██╗██╔══██╗██╔══██╗
  ███╔╝ ███████║██████╔╝██████╔╝███████║
 ███╔╝  ██╔══██║██╔═══╝ ██╔═══╝ ██╔══██║
███████╗██║  ██║██║     ██║     ██║  ██║
╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝  ╚═╝

Welcome to Zappa!

Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!

# zappa 화경 이름 설정 
Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'): rams_zappa_test

# 위에서 설정한 AWS credential에서 데이터를 가져 올 수 있습니다.  
AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
We found the following profiles: eb-cli, default, and zappa. Which would you like us to use? (default 'default'): zappa

# S3버킷의 이름을 설정 해 줍니다. 
Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
What do you want to call your bucket? (default 'zappa-o4vpxzdhk'): zappa-rams-test

# Django settings의 경로 셋팅 
It looks like this is a Django application!
What is the module path to your projects's Django settings?
We discovered: config.settings
Where are your project's settings? (default 'config.settings'): config.settings

# n로 진행 했습니다. 
You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: n

Okay, here's your zappa_settings.json:

{
    "rams_zappa_test": {
        "django_settings": "config.settings",
        "profile_name": "zappa",
        "project_name": "zappa_test_project_name",
        "runtime": "python3.8",
        "s3_bucket": "zappa-rams-test"
    }
}

Does this look okay? (default 'y') [y/n]: y

Done! Now you can deploy your Zappa application by executing:

        $ zappa deploy rams_zappa_test

After that, you can update your application code with:

        $ zappa update rams_zappa_test

To learn more, check out our project page on GitHub here: https://github.com/Miserlou/Zappa
and stop by our Slack channel here: https://slack.zappa.io

Enjoy!,
 ~ Team Zappa!
```

4. region 추가 

위 과정을 거치면, 프로젝트의 루트에 `zappa_settings.json`가 생성 됩니다. 

해당 파일을 통해 zappa의 여러 환경 설정을 진행 할 수 있습니다. 

[Zappa 공식 Github](https://github.com/Miserlou/Zappa#advanced-settings)에서 구체적인 설정 내용들을 확인 할 수 있습니다. 

여기서 `aws_region` 항목을 `ap-northeast-2`로 추가 해주어 `서울`지역으로 설정 해 줍니다. ( `ap-northeast-2`는 서울을 의미합니다. 다른 지역은 [AWS 공식 문서](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)를 참조 해주세요 )

5. Zappa 배포

```bash
$ zappa deploy
```

[공식 GIthub](https://github.com/Miserlou/Zappa#advanced-settings)에 따르면, 위 과정을 거친 뒤 `zappa deploy`만 하면, 배포가 진행 된다고 적혀 있지만 실제로는 많은 오류가 있어 배포가 되지 않습니다.  

오류에 대응하기 위한 추가적인 설정에 대해서는 다음 포스팅에 추가적으로 서술 하겠습니다. 



