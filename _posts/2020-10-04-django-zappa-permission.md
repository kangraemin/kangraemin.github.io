---
title: Zappa를 이용하여 AWS Lambda에 Django 배포하기 - Add IAM Permission (2)
date: 2020-10-04
categories:
 - Django
tags:
 - AWS
 - Zappa
 - Server deployment
 - Django
 - API Gateway
 - Lambda
---

Zappa를 활용하여 Django 프로젝트를 API Gateway + Lambda를 활용한 서버리스 아키텍쳐로 배포하는 과정에 대해 공유합니다.    

이번 글에서는 zappa deploy 도중 겪었던 에러들과 해결 과정을 공유합니다. 

<!-- more -->

## IAM role 관련 에러

```bash
$zappa deploy rams_zappa_test

Calling deploy for stage rams_zappa_test..
Creating zappa_test_project_name-rams-zappa-test-ZappaLambdaExecutionRole IAM Role..
Error: Failed to manage IAM roles!
You may lack the necessary AWS permissions to automatically manage a Zappa execution role.
Exception reported by AWS:An error occurred (AccessDenied) when calling the CreateRole operation: User: arn:aws:iam::301412762219:user/rams is not authorized to perform: iam:CreateRole on resource: arn:aws:iam::301412762219:role/zappa_test_project_name-rams-zappa-test-ZappaLambdaExecutionRole
To fix this, see here: https://github.com/Miserlou/Zappa#custom-aws-iam-roles-and-policies-for-deployment
```

IAM 계정의 퍼미션을 제대로 주지 않았다면, 위와 같이 deploy 도중 permission 관련 에러가 뜰 수 있습니다.

API Gateway, lambda등의 설정들에 대해 zappa에서 설정 해 주어야 할 것들이 있는데, zappa에게 준 IAM계정이 그럴 권한이 없을 때 생기는 문제 입니다. 

관련 내용은 [Zappa 공식 Github](https://github.com/Miserlou/Zappa#using-custom-aws-iam-roles-and-policies)에서 찾아 볼 수 있습니다.

위 문서에서, [AWS polices Git issue](https://github.com/Miserlou/Zappa/issues/244)를 소개 하고 있습니다. 

### Permission 추가 방법 

![pic1.png](/assets/images/posts/2020-10-04-django-zappa-permission/pic1.png)

AWS 콘솔에 접속 및 IAM 서비스 콘솔로 이동 후 Users 항목으로 이동 하면 위의 화면을 볼 수 있습니다. 

![pic2.png](/assets/images/posts/2020-10-04-django-zappa-permission/pic2.png)

여기서 Add permission을 누르고 `Attach existing policies directly`클릭 후 `Create policy`클릭 

![pic3.png](/assets/images/posts/2020-10-04-django-zappa-permission/pic3.png)

이후 나오는 화면에서, JSON 클릭 후 원하는 Permission을 입력하고 추가적인 진행을 해주면 됩니다. 

저는 위에 적은 [AWS polices Git issue](https://github.com/Miserlou/Zappa/issues/244)를 참고하여 아래와 같이 IAM에 permission을 추가 해주었습니다. 

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:AttachRolePolicy",
                "iam:GetRole",
                "iam:CreateRole",
                "iam:PassRole",
                "iam:PutRolePolicy"
            ],
            "Resource": [
                "arn:aws:iam::[본인의 iam account 번호]:role/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "lambda:CreateFunction",
                "lambda:ListVersionsByFunction",
                "lambda:DeleteFunctionConcurrency",
                "logs:DescribeLogStreams",
                "events:PutRule",
                "lambda:GetFunctionConfiguration",
                "cloudformation:DescribeStackResource",
                "apigateway:DELETE",
                "apigateway:UpdateRestApiPolicy",
                "events:ListRuleNamesByTarget",
                "apigateway:PATCH",
                "events:ListRules",
                "cloudformation:UpdateStack",
                "lambda:DeleteFunction",
                "events:RemoveTargets",
                "logs:FilterLogEvents",
                "apigateway:GET",
                "lambda:GetAlias",
                "events:ListTargetsByRule",
                "cloudformation:ListStackResources",
                "events:DescribeRule",
                "logs:DeleteLogGroup",
                "apigateway:PUT",
                "lambda:InvokeFunction",
                "lambda:GetFunction",
                "lambda:UpdateFunctionConfiguration",
                "cloudformation:DescribeStacks",
                "lambda:UpdateFunctionCode",
                "events:DeleteRule",
                "events:PutTargets",
                "lambda:AddPermission",
                "cloudformation:CreateStack",
                "cloudformation:DeleteStack",
                "apigateway:POST",
                "lambda:RemovePermission",
                "lambda:GetPolicy"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucketMultipartUploads",
                "s3:CreateBucket",
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:AbortMultipartUpload",
                "s3:DeleteObject",
                "s3:ListMultipartUploadParts"
            ],
            "Resource": "arn:aws:s3:::*"
        }
    ]
}
```

이후 `zappa deploy`를 다시 진행하면, permission 에러는 더이상 뜨지 않는것을 확인 할 수 있습니다. 
