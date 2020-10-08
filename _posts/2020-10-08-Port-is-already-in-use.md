---
title: That port is already in use in macOS
date: 2020-10-08
categories:
 - Django
tags:
 - Django
 - MacOS
---

macOS에서 `That port is already in use`에러를 해결하는 방법을 공유합니다. 

<!-- more -->

Django 서버를 구동 하거나, Jekyll local 빌드를 하는 경우에 자주 발생하던 오류입니다. 

해당 포트로 이미 서버를 가동중인데, 해당 포트에 다른 서버를 빌드하고자 할 때 나는 오류입니다.

간단한 오류이지만, 해결 방법을 자주 까먹게 되어 블로그에 정리하고자 합니다. 

아래의 명령어를 사용하여 (해당 명령어에서는 8000번 포트) 해당 포트의 작업을 종료 시켜 주면 문제가 해결 됩니다.

```bash
sudo lsof -t -i tcp:8000 | xargs kill -9
```