---
title: Android Cook Book (2) - Git / Github / SourceTree를 활용하여 안드로이드 코드 관리하기
date: 2020-07-25
categories:
 - Android
tags:
 - Android
 - Android Lecture
 - Toy project
 - Git
 - Github
 - SourceTree
---

안드로이드 어플 개발을 시작하기 위한 Cook book 입니다. 앱을 만들고는 싶은데, 어떻게 시작해야 할 지 모르는 분들이 제 포스팅을 따라 공부 한 뒤, 그 다음부턴 스스로 앱개발을 주저없이 하실 수 있도록 하는것이 저의 목표입니다. 개발에 대한 사전지식 없이 바로 일정 관리 앱을 만들어 보며 개발을 시작 할 것이며 개발을 진행하다 필요한 것은 필요 할 때 방향을 잡는 방향으로 진행 할 것입니다.

<!-- more -->

> TL;DR (Too Long Didn’t Read)
> <br><a class="nav-link" href="#github-회원가입-및-저장소-만들기"><span class="nav-text">Github 회원가입 및 저장소 만들기</span></a> 부터 확인 해주시면 됩니다.
> <br>잘못된 부분, 추가적인 설명이 필요한 부분이 있으면 꼭 지적 부탁드립니다 !

### Source Code 관리의 필요성

 자, 우리는 [지난 강좌](https://kangraemin.github.io/android/2020/07/23/Make-android-project/) 에서 첫 Android Project를 각 개인의 컴퓨터에 생성 하였습니다. 그리고 여러분이 안드로이드 개발을 진행 할 때 코드들은 각 개인의 컴퓨터에 저장 될 것입니다. 

 하지만 각 개인의 컴퓨터에**만** 코드를 저장 하는것은 몇가지 문제점이 존재합니다.

1. 개인의 컴퓨터에 문제가 생겨 코드가 손실 될 위험이 있습니다. 
2. 코드가 자주 수정 되기 때문에 내가 과거에 짰던 코드로 코드를 복구시키는 일이 쉽지 않습니다. 
3. 내가 짠 코드를 타인과 공유하는데 어려움이 있습니다. 
    - 혼자 개발 할 때엔 코드를 타인과 공유하는일이 흔치 않은 일일 수 있습니다. 하지만, 회사에서 한 프로젝트를 한명이 모두 개발하는 경우는 흔치 않아 협업을 진행 해야 하는 경우가 **매우**흔합니다.
    - 협업을 할 떄 내가 짠 코드를 타인과 공유를 해야 하며, 각 개인의 컴퓨터에만 서로의 코드가 짜져 있다면, 서로의 코드를 공유하는것이 쉽지 않습니다.

앞에서 말씀드린 세가지 이외에도 여러 문제점들이 존재하며, 이 문제는 여러분들 이외의 다른 개발자들도 고민 했던 문제였습니다. `Git` 은 이러한 문제를 해결 하기 위해 만들어진 소프트웨어 입니다. 코드 백업을 도와주는 소프트웨어라고 생각하시면 편할 것 같아요. 

`Git`을 통해 코드를 로컬에 백업을 하면, `.git` 이라는 폴더가 생성되며 백업 내용들은 `.git`이라는 폴더에 저장됩니다. 우리는 해당 폴더에 언제든지 백업 할 수 있으며, 백업 내용을 언제든지 가져 올 수 있습니다. 

하지만 `.git` 코드 백업 내용을 타인과 어떻게 공유 하는지가 문제입니다. Github는 이러한 문제점을 해결 하기 위해 코드 백업 내용들을 **로컬**에 저장하는 것이 아닌 **웹**에 저장할 수 있도록 도와주는 웹 서비스 중 한가지 입니다.

SourceTree란, `git`을 활용하여 `Github`에 쉽게 코드를 백업 할 수 있도록 도와주는 프로그램 중에 한가지 입니다. 

정리하자면

- Git을 활용하여 우리는 프로젝트의 코드를 백업하여 쉽게 관리 할 수 있습니다.
- Github는 백업 내용을 로컬에 저장하는 것이 아닌 웹에 저장하도록 도와주는 웹 서비스 중 한가지 입니다.
- SourceTree는 백업 내용을 웹에 저장하기 쉽게 도와주는 프로그램 중 한가지 입니다.

 ( Git과 Github에 대한 내용을 잘 설명해놓은 동영상이 있어 소개드립니다. [https://www.youtube.com/watch?v=Bd35Ze7-dIw](https://www.youtube.com/watch?v=Bd35Ze7-dIw) )

그래서 이번 글에서는, Git을 통해 프로젝트의 소스코드를 Github 라는 웹페이지에 SourceTree를 활용하여 업로드 하는 방법을 알아 볼 것입니다. 

### Github 회원가입 및 저장소 만들기

![pic1.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic1.png)<center><a href="https://github.com/">https://github.com/</a>에 접속합니다.</center>

![pic2.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic2.png)<center>회원 가입을 진행 한 뒤, 오른쪽 상단의 아이콘을 클릭하여 자신의 이름 (빨간색 네모 부분)을 클릭 해줍니다.</center>

![pic3.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic3.png)<center>Repository를 클릭 해 줍니다. 여기서 Repository란, 말 그대로 코드를 저장하는 공간을 의미하며 Repo 로 줄여 쓰기도 합니다. </center>

![pic4.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic4.png)<center>New를 클릭하여, 앞으로 안드로이드 코드를 저장 할 새로운 Repository를 생성 해줍니다.</center>

![pic6.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic6.png)<center>Repo를 생성 한 직후입니다. 아무 코드도 저장되어 있지 않은 경우 해당 사진처럼 표시됩니다.</center>

### Download Git & SourceTree 

![pic8.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic8.png)<center><a href="https://git-scm.com">https://git-scm.com</a>에 접속하여 Git을 다운받아 줍니다.</center>

### Download SourceTree

제가 이미 SourceTree가 설치되어 있어 SourceTree를 다운받는 방법은 다른 블로그를 참조하여 다운받아주세요. 

### Git init project

![pic9.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic9.png)<center>Android Studio 에서 오른쪽 아래, Terminal을 눌러줍니다.</center>

Terminal에서 아래와 같이 `git --version`을 입력하여 git이 설치 되었는지 여부를 확인합니다.

```bash
git --version
>> git version 2.24.3 (Apple Git-128)
```

또 아래와 같이 `git init`을 입력하여, git 로컬 저장소를 초기화 해줍니다. 

```bash
git init
>> Initialized empty Git repository in /Users/rmkang/AndroidStudioProjects/RaeminToDoApp/.git/
```

### Connect with Github repo

![pic10.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic10.png)<center>소스트리에서, 로컬 저장소를 가져 옵니다.</center>

![pic11.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic11.png)<center>가져온 로컬저장소를 더블 클릭 한 뒤, 오른쪽 상단에 있는 `설정`을 누르고 `원격`을 눌러 줍니다.</center>

![pic12.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic12.png)<center>`URL / 경로` 에 생성 한 저장소의 git 주소( repo url 주소 (https://github.com/kangraemin/Project_ToDo) + .git )를 입력 한 뒤 확인 을 누릅니다.</center>

![pic13.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic13.png)<center>스테이지에 올라가지 않은 파일은, 현재 로컬 저장소에 제가 작업 한 내용 중 원격 저장소로 올리지 않을 내용이고, 스테이지에 올라간 파일은 원격 저장소로 업로드 할 내용입니다. 안드로이드 프로젝트 전체를 업로드 할 것이기 때문에 모든 파일을 스테이지에 올려주고, 커밋 메세지를 적절한 메세지 ( 저같은 경우는 first commit )으로 작성 해준 뒤 커밋 버튼을 눌러줍니다.</center>

![pic14.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic14.png)<center>왼쪽 상단 ( 11시 방향 )의 푸쉬를 눌러, 커밋 했던 내용을 원격 저장소에 업로드 해 줍니다.</center>

![pic15.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic15.png)<center>Github에 들어가 저장소 ( 만들었던 저장소 )에 들어가 다음과 같이 파일이 업로드 된것이 확인된다면 깃허브에 업로드를 완료 한 것입니다 !</center>

![pic16.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic16.png)<center>여러분 개인 계정 화면으로 가셔셔 초록색 네모가 생겨난 것을 확인 해주세요 ! 하루에 깃허브 활동 ( 커밋 등 )을 통해 코드에 기여한 횟수를 알려줍니다. 해당 초록색 공간을 보통 `잔디`라고 부르는데, 잔디가 쌓여가는걸 보며 성취감을 느껴보세요 ㅎㅎㅎ</center>

<center>오늘은 여기까지 하도록 하고, 잘못된 부분, 추가적인 설명이 필요한 부분이 있으면 꼭 댓글로 지적 부탁드립니다 !</center>

2020-07-27 수정

위에 보니 코틀린으로 프로젝트를 생성했었네요 ... 위에서 생성한 repo 삭제하고, 다시 repo 만들어 주고 안드로이드 프로젝트 자바로 생성 한 뒤 위와 같은 프로세스를 똑같이 다시 진행해주었습니다 ~~ 

![pic17.png](/assets/images/posts/2020-07-25-Git-Github-SourceTree/pic17.png)