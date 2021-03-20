---
title: IntelliJ 기반 IDE ( Android Studio / PyCharm / IntelliJ ) - 유용한 단축키 모음 
date: 2021-03-18
categories:
 - Android
tags:
 - Android Studio
 - IntelliJ
 - PyCharm
 - Keymap
---

IntelliJ 기반의 IDE ( Android Studio / PyCharm / IntelliJ ... ) 등에서 정말 유용하게 사용하고 있는 단축키를 소개합니다. 

해당 포스팅은 macOS의 Android Studio를 기반으로 작성하였습니다. 

<!-- more -->

아래에 소개된 단축키들은 개인적으로 어떤 것을 더 많이 사용하고 적게 사용하고 판단 할 수 없을정도로 정말 많이 사용하고 있는 단축키들 입니다. 

한번쯤 써보시는걸 꼭 추천드립니다

1. Add Selection for Next Occurrence

Keymap의 `Main menu | Edit | Find | Add Selection for Next Occurrence` 항목 이며, macOS에서는 `control` + `G` 이며, 저는 vsCode와 동일하게 사용하기 위해 `cmd` + `D`로 변경 한 뒤 사용하고 있습니다. 

현재 선택하고 있는 블록과 동일한 텍스트를 찾아 선택 해 줍니다. 

선택할 때 대소문자, 띄어쓰기를 구분합니다. 

![1-next-occurence.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/1-next-occurence.gif)

![2-next-occurence-2.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/2-next-occurence-2.gif)

정말 많은 용도로 사용하고 있지만, 위와 같이 xml의 id 값을 선택하여 Kotlin / Java 코드에 가져올 때 정말 편하게 사용 할 수 있습니다. 

2. Search everywhere

Keymap의 `Main menu | Navigate | Search Everywhere` 항목 이며, macOS에서는 `shift` + `shift`로 할당되어 있습니다. 

![3-shift-twice.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/3-shift-twice.gif)

원하는 기능에 대한 이름만 알고 있으면, 어떤 것이든 검색 해주는 기능입니다. 

해당 기능을 통해 Android Studio의 옵션을 킬 수도 있고, 파일 명으로 파일을 검색하여 이동 할 수도 있습니다. 

3. Camel Case plugin / Clone Caret Below

해당 기능은 Keymap에 들어가있는 기능은 아니지만, 정말 유용하게 쓰고 있는 기능이라 공유 합니다. 

선택한 단어를 kebab-case, snake_case, camelCase 등의 형식으로 변경 해 주는 플러그인입니다. [camelCase plugin](https://plugins.jetbrains.com/plugin/7160-camelcase)

`shift`를 두번 눌러, Search everywhere 창을 띄운 후 `plugins`를 검색 한뒤 들어 가 보시면 IDE의 plugin 설정창이 나옵니다.

![camel-case-plugin.png](/assets/images/posts/2021-03-18-IntelliJ-keymap/camel-case-plugin.png)

camelCase 플러그인을 설치 해 주시고, 단어를 선택 한 뒤 `option` + `shift` + `u`를 눌러 보시면, 해당 단어들이 다른 형식의 단어 포맷으로 변경되는것을 확인 하실 수 있습니다. 

![4-camel-case.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/4-camel-case.gif)

![5-camel-case-2.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/5-camel-case-2.gif)

해당 플러그인을 통해, xml에서 가져온 id 값들이 snake_case로 되어 있다면, 해당 id 값들을 편하게 camelCase로 변경하여 사용 할 수 있습니다. 

또 위 gif에서, 커서를 아래로 하나씩 복사 하는 것을 확인 할 수 있는데요

이는 Keymap의 `Editor Actions | Clone Caret Below` 항목이며 macOS에서는 `option` 두번 + `up` / `down` 으로 할당되어 있습니다. 

저는 vsCode와 동일하게 사용하기 위해 `cmd` + `option` + `up` / `down`으로 할당하여 쓰고 있습니다. 

4. Reformat code 

Keymap의 `Main menu | Code | Reformat Code` 항목이며, macOS에서는 `option` + `cmd` + `l`로 할당되어 있습니다. 

![6-reformat-xml.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/6-reformat-xml.gif)
![7-reformat-kotlin.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/7-reformat-kotlin.gif)

IDE에 설정되어 있는 코드 스타일 대로 코드를 알맞게 정리 해주는 기능입니다. 

개인적으로 작업을 마무리 하기 전 반드시 사용하는 단축 키 중에 하나입니다. 

5. Extend Selection

Keymap의 `Editor Actions | Extend Selection`항목이며, macOS에서는 `option` + `up`로 할당되어 있습니다. 

![8-extend-selection.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/8-extend-selection.gif)

단어 -> 코드 한줄 -> 함수 전체 등 커서에서 부터 범위가 점점 커지면서 선택 해주는 단축키 입니다. 

6. Optimize imports

Keymap의 `Main menu | Code | Optimize Imports`항목이며, macOS에서는 `ctrl` + `option` + `o`로 할당되어 있습니다. 

![9-optimize-imports.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/9-optimize-imports.gif)

사용되지 않고 있는 import 삭제, import를 IDE 설정에 맞게 자동으로 정리 해 주는 등 import를 최적화 해주는 단축키 입니다. 

하지만 해당 단축키는 현재 유저의 커서가 올라가있는 파일의 import만 최적화 해 주는데, project 전체 파일을 검사 한 뒤 최적화가 필요한 파일들에 대해 한번에 최적화를 해주는 방식도 있습니다. 

![10-run-inspection-optimize-impor.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/10-run-inspection-optimize-impor.gif)

`shift`를 두번 눌러, Search everywhere 창을 띄운 후 `run inspection by name`을 검색하시고, `unused import directly` ( kotlin 파일 import 검사 ), `unused import` ( JAVA 파일 import 검사 )를 통해 위와 같이 import 최적화를 한번에 할 수 있습니다. 

개인적으로 작업을 마무리 하기 전 반드시 사용하는 기능 중에 하나입니다. 

7. Extract function 

Keymap의 `Main menu | Refactor | Extract | Extract Method`항목이며, macOS에서는 `option` + `cmd` + `m`으로 할당되어 있습니다. 

![11-extract-function.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/11-extract-function.gif)

선택한 코드들을 함수로 추출 해 낼 수 있습니다. 

8. Rename

Keymap의 `Main menu | Refactor | Rename`항목 이며, macOS에서는 `shift` + `F6`로 할당되어 있습니다. 

![12-rename.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/12-rename.gif)

변수, 클래스, interface 등의 이름을 편하게 바꾸어 주는 단축키 이며, 해당 변수가 사용되고 참조되고 있는 모든 곳의 이름이 변경 됩니다. 

9. Implement Methods

Keymap의 `Main menu | Code | Implement Methods`항목 이며, macOS에서는 `control` + `i`로 할당되어 있습니다.  

![13-impl.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/13-impl.gif)

interface, abstract class 등을 구현, 상속 받아 사용 할 때 반드시 구현 되어야 하는 메소드, property가 있으면 쉽게 구현 할 수 있도록 추가 해 줍니다. 

10. Go to Implementation(s)

Keymap의 `Main menu | Navigate | Go to Implementation(s)`항목 이며, macOS에서는 `option` + `cmd` + `b`로 할당되어 있습니다. 

![14-move-to-imp.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/14-move-to-imp.gif)

메소드에 커서를 올려놓고, `cmd` + `b`를 누르면 메소드가 사용되고 있는 곳이나, 메소드의 선언부로 이동 할 수 있습니다. 

하지만 interface 구현 객체의 method에 `cmd` + `b`는 interface의 선언부로 이동이 되기 때문에, 함수의 구현체로 가는데 어려움이 있습니다. (귀찮습니다) 

이때 `option` + `cmd` + `b`를 사용한다면, 인터페이스, 추상 클래스 안에 있는 함수의 구현체로 바로 이동 할 수 있습니다. 

인터페이스를 자주 사용하는 프로젝트에서 정말 유용하게 사용하실 수 있습니다. 

11. Move to Last Edit Location

Keymap의 `Main menu | Navigate | Last Edit Location`항목이며, macOS에서는 `cmd` + `shift` + `backspace`로 할당되어 있습니다.

![15-latest-modified.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/15-latest-modified.gif)

최근에 코드를 수정 한 위치로 바로 이동 할 수 있습니다. 

최근 코드 수정 위치가 현재 파일의 위치와 다르더라도, 코드 수정 위치가 존재하는 파일로 바로 이동됩니다. 

12. Move to Last Cursor Location

Keymap의 `Main menu | Navigate | Back`항목이며, macOS에서는 `cmd` + `option` + `left` / `right`로 할당되어 있습니다. 

![16-latest-cursor.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/16-latest-cursor.gif)

가장 최근에 내가 이동했던 커서 위치로 이동 할 수 있습니다. 

`left`를 누르면 최근 -> 예전 순으로, `right`는 예전 -> 최근 순으로 이동합니다. 

13. Go to File

Keymap의 `Main menu | Navigate | Go to File`항목이며, macOS에서는 `cmd` + `shift` + `o`로 할당되어 있습니다. 

![17-open-file.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/17-open-file.gif)

프로젝트에 존재하는 파일을 파일 이름으로 검색 후 접근 할 수 있도록 해주는 단축키 입니다. 

14. Next Highlighted Error

Keymap의 `Main menu | Navigate | Next Highlighted Error`항목이며, macOS에서는 `F2`로 할당되어 있습니다. 

![18-find-error.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/18-find-error.gif)

현재 파일에 컴파일 에러가 있다면, 에러 위치로 바로 이동시켜주는 단축키 입니다. 

15. Related Symbol

Keymap의 `Main menu | Navigate | Related Symbol`항목이며, macOS에서는 `control` + `cmd` + `up`으로 할당되어 있습니다. 

![19-related-symbol.gif](/assets/images/posts/2021-03-18-IntelliJ-keymap/19-related-symbol.gif)

안드로이드에서 주로 사용하고 있으며 현재 위치하고 있는 자바 / Kotlin 파일에 연관된 Xml파일로 쉽게 이동 할 수 있습니다. 







