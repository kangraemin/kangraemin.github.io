---
title: iOS - image set에서 1x, 2x, 3x 이미지를 사용하는 이유
date: 2021-01-21
categories:
 - IOS
tags:
 - IOS
---

> 픽셀 / 포인트 개념을 바탕으로 iOS 프로젝트에 png 이미지를 추가할 때 image set의 1x, 2x, 3x 이미지에 대해서 공부 한 것을 공유합니다.

<!-- more -->

## 왜 iOS에선 3가지 형식( 1x / 2x / 3x )의 png 파일이 필요한가

그 이유는에 대해서는 point (pt) / pixel (px)에 대해서 알아 볼 필요가 있습니다. 

### 픽셀 

먼저 픽셀(화소)이란, Picture Element의 합성어이며 디스플레이를 구성하고 있는 최소 단위를 뜻하고, px이라 축약해서 사용합니다.

화면의 단위 면적에, 픽셀이 많이 밀집되어 있으면 있을수록 화면은 선명하게 나타납니다. 

참고로 아이폰 1세대의 경우, 디스플레이로 HVGA 해상도 (480×320 = 153,600 화소)의 디스플레이를 사용하였습니다.

또 이 픽셀은, 디자이너가 개발자에게 뷰의 크기를 알려 줄 때 사용되기도 했습니다.

예를 들어, 디자이너님이 버튼을 디자인하고, 해당 버튼의 크기를 개발자에게 가로 20픽셀, 세로 10픽셀로 알려 주는 경우도 있었습니다. 

### 포인트

그리고 디스플레이 기술이 발전함에 따라 같은 면적에 픽셀을 더 많이 추가하여 매끄러운 화면을 만들 수 있게 되었습니다. 

하지만, 같은 면적에 픽셀 수가 늘어나면, 가로 20픽셀, 세로 10픽셀 인 버튼이, 해상도가 높은 기계에선 같은 하드웨어 면적에 픽셀 수가 더 늘어나기 때문에 디자이너가 의도했던 것 보다 작게 보일 수 밖에 없습니다. 

아이폰 4세대 발매에는, 애플이 레티나 디스플레이를 선보이며 위 문제를 해결하기 위해 포인트라는 개념을 도입 하였습니다. 

포인트는, 1인치를 72로 나눈 값이 1 포인트 입니다. ( 절대적인 값이며, 약 0.3527mm에 해당합니다. )

iOS 하드웨어 기기가 초창기 ( 아이폰 4세대 이전 기기 )에 나올 때는, 1포인트당 1픽셀로 동일 하여 포인트라는 개념이 필요가 없었지만, 아래의 사진과 같이 4세대 (레티나 디스플레이) 부터는 1포인트 당 4픽셀이 들어가게 됩니다. 

심지어, 아이폰 6 plus 부터는 retina HD display를 사용하여 1포인트당 9 픽셀이 들어가게 됩니다.

![pic1.png](/assets/images/posts/2021-01-21-iOS-image-set/pic1.png)
출처 : [https://blog.fluidui.com/designing-for-mobile-101-pixels-points-and-resolutions/](https://blog.fluidui.com/designing-for-mobile-101-pixels-points-and-resolutions/)

따라서, 유저는 전보다 훨씬 깔끔해진 화면을 볼 수 있는 대신에, 디자이너들은 더이상 픽셀 단위로 화면을 개발자들에게 제공 할 수 없었고, 이를 위해 포인트 단위로 화면을 개발자들에게 제공하였습니다. 

포인트 단위로 화면의 크기를 제시함으로써, ppi ( pixel per inch )에 관계 없이 같은 크기의 화면을 볼 수 있게 되었습니다. 

### 1x / 2x / 3x

그래서, png파일을 iOS 프로젝트에서 image set으로 사용하고자 할 때, 왜 1x / 2x / 3x 세가지를 넣어야 하는걸까요

위에서 이야기 했듯, 표준 디스플레이 -> retina 디스플레이 -> retina HD 디스플레이 로 디스플레이가 업그레이드 될 때마다, 1포인트당 들어가는 픽셀 수는 1개 -> 4개 -> 9개로 늘어났습니다.

즉, 고해상도 디스플레이를 사용하는 기기일수록 단위 면적당 들어가는 픽셀 수가 더 많기 때문에 고해상도 디스플레이에는 고해상도 사진이 들어가야만 이미지가 깨지지 (계단 형태로 흐리게 보이지) 않습니다. 

![pic2.png](/assets/images/posts/2021-01-21-iOS-image-set/pic2.png)
출처 : [https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/image-size-and-resolution/](https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/image-size-and-resolution/)

위 사진에서 확인해본다면, retina HD 디스플레이를 사용하는 기기에서 픽셀 수가 낮은 이미지를 사용하게 된다면, 위 사진의 제일 왼쪽과 같은 이미지가 보일것입니다. 

하지만 표준 디스플레이를 사용하는 기기에서, 픽셀수 가높은 이미지를 사용한다고 해서 하드웨어 기기의 픽셀 수가 적기 때문에 위 사진의 제일 오른쪽과 같은 이미지가 보이진 않습니다.

따라서, 표준 디스플레이를 사용하는 기기에서는, 1포인트당 1픽셀이 들어가는 1x 이미지가 사용되고 retina 디스플레이를 사용하는 기기에서는 1포인트당 4픽셀이 들어가는 2x 이미지가 사용되며 retina HD 디스플레이를 사용하는 기기에서는 1포인트당 9픽셀이 들어가는 3x 이미지가 사용되어야 이미지가 올바르게 출력됩니다.

iOS 개발자는, 유저가 어떤 기기를 사용하게 될 지 알 수 없으므로 모든 유저에게 알맞은 이미지를 제공하기 위해 1x, 2x, 3x 이미지를 준비하여 image set으로 사용하는 것입니다. 

> 개인적으로 공부 중에 작성한 포스트 입니다. 틀린 점 있으면 꼭 지적 부탁드립니다. 