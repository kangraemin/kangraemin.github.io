---
title: iOS 하드웨어 기기에 앱 무선으로 빌드하기
date: 2020-10-07
categories:
 - IOS
tags:
 - IOS
 - Xcode
---

[이전 글](https://kangraemin.github.io/ios/2020/10/06/iOS-build/)에서 이어지는 내용입니다.   

Xcode에서 하드웨어 아이폰 기기에 무선으로 빌드하는 과정을 공유합니다. 

<!-- more -->

## 무선으로 iOS 앱 빌드 

이전 글은 **유선 케이블로 iOS에 연결 한 후**에 **무선**으로 Xcode와 iOS기기를 연결 하는 과정입니다. 

**반드시** 한번은 연결이 되어야 하므로 유선으로 iOS 기기를 한번도 연결 하지 않으신 분은 [이전 글](https://kangraemin.github.io/ios/2020/10/06/iOS-build/)을 참고 해주세요 

> 꼭 iOS 기기와, 노트북이 같은 wifi 환경에 있어야 연결 가능하다는 것을 주의 해주세요 

### 1. 아래 그림 처럼 Xcode의 `window` -> `Devices and simulators` 항목을 선택합니다. 

![pic1.png](/assets/images/posts/2020-10-07-iOS-wireless-build/pic1.png)

### 2. 아래 사진에 보이는 `Connect Via Network`를 체크 해 줍니다. 

![pic2.png](/assets/images/posts/2020-10-07-iOS-wireless-build/pic2.png

그 뒤 빌드를 눌러 주시면 무선으로 빌드가 되는것을 확인 할 수 있습니다. 