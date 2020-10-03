---
title: Zappa를 이용하여 AWS Lambda에 Django 배포하기
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

서버리스의 단어 그대로를 직역 해보자면, `서버가 없는`으로 해석 될 수 있습니다. 

하지만, 실제 서버가 없이 클라이언트-서버 시스템을 구현 할 수는 없습니다. 

따라서 서버리스도 실제 서버가 존재하지 않는 것은 아닙니다. 

서버리스란, 서버 관리를 제가 직접 하는 것이 아니라 관리를 직

## Zappa 

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



