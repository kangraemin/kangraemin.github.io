---
title: Sequel Pro / MySQLWorkbench에서 MySQL RDS 접속하기
date: 2020-10-05
categories:
 - AWS
tags:
 - AWS
 - Server deployment
 - RDS
---

RDS 생성 이후에, Sequel Pro나 MySQLWorkbench 등 DB 관리 툴에서 RDS에 접속하는 도중 겪었던 문제를 공유합니다. 

<!-- more -->

## RDS 생성

AWS RDS를 생성 하신 이후에, 해당 DB안에 있는 내용을 보기 위해 Sequel Pro나 MySQLWorkbench를 사용 할 수 있습니다. 

> 참고 : [Sequel Pro 공식 홈페이지](https://www.sequelpro.com/), [MySQLWorkbench 공식 홈페이지](https://www.mysql.com/products/workbench/)

Host, Password, Port 등에 오타가 없는데도 외부 프로그램에서 RDS에 접근 할 수 없는 경우가 있습니다. 

이를 위해 RDS의 보안 그룹의 인바운드 규칙에 본인의 IP나 특정 접속하고자 하는 IP를 추가 해 줄 필요가 있습니다.

위 과정은 [앞의 글](https://kangraemin.github.io/aws/2020/09/27/elasticbeanstalk-postgrsql/)에 서술 되어 있는데, 앞의 글에서는 RDS가 해당 보안그룹에 속하도록 설정 해 준것이고, 내 IP를 추가 하기 위해선 `사용자 지정`대신 `내 IP`를 선택 해 주면 됩니다. 

하지만, RDS의 보안 그룹의 인바운드 규칙에 본인 IP나 외부의 특정 IP를 허용 해 준다고 하더라도 접속 요청이 TimeOut이 되며 접속이 되지 않는 경우가 있습니다. 

이 경우엔 RDS 자체 설정에서 Public 접근을 막은 것 때문 일 수 있습니다. 

RDS 인스턴스 콘솔 - Connectivity & security - Public accessibility 항목을 확인 해주시고, 해당 항목이 No 라면 Yes로 바꿔 주어야 합니다. 

RDS 콘솔에서 Modify를 누르시고, 아래 사진과 같이 수정 해주신 뒤 즉석 적용을 선택 하시면 약 5분 정도 뒤부터 접속이 가능 해집니다. 

![pic1.png](/assets/images/posts/2020-10-05-RDS-public-permission/pic1.png)

