---
title: 운영체제 스터디 (9) - 가상 메모리의 기초
date: 2021-03-08
categories:
 - Operation System
tags:
 - OS
 - Computer structure
 - Computer science
---

> 운영체제 스터디 도중 [쉽게 배우는 운영체제](http://www.yes24.com/Product/Goods/62054527)를 읽고 요약한 내용입니다. 자세한 내용은 책을 구매하여 확인 부탁드립니다. 

<!-- more -->

# 8. 가상 메모리의 기초

## 가상 메모리의 개요

### 가상 메모리 시스템

- 물리 메모리의 크기와, 프로세스가 올라갈 메모리의 위치를 신경 쓰지 않고 프로그래밍 할 수 있도록 지원 하는 것
- 물리 메모리 크기와 상관 없이 프로세스에 커다란 메모리 공간을 제공하는 기술
- 가상 메모리 시스템의 모든 프로세스는, 물리 메모리와 별개로 자신이 메모리의 어느 위치에 있는지 상관 없이 0번지 부터 시작하는 연속된 메모리 공간을 가짐
- 논리 주소
    - 물리 메모리의 주소 공간에 비례
- 가상 주소
    - 물리 메모리 공간이 아닌 가상의 주소 공간을 가짐
- 가상 메모리는, 이론적으론 무한대의 크기 이지만, 현실적인 최대 크기는 물리 메모리의 최대 크기로 한정 된댜 ( 32Bit CPU → 4GB ... )
- 또, 물리 적으로 가상메모리에서 사용 할 수 있는 메모리의 전체 크기는 물리 메모리 크기 + 스왑 영역의 크기 이다.
- 동적 주소 변환 → 프로세스가 사용하는 가상 주소를 실제 메모리의 물리 주소로 변환 하는 작업
- 가상 메모리 에서는, 메모리 분할 방식으로 세그멘테이션-페이징 혼용 기법을 주로 사용 한다.

![가상메모리, 물리메모리](/assets/images/posts/2021-03-08-virtual-memory/pic1.png)

### 매핑 테이블

- 음식점의 대기 손님에 대해, 번호 / 인원 / 주문 등을 적은 테이블을 만들 어 놓으면 자리 배정과 주문이 원할하게 이루어 질 수 있음
- 메모리를 관리 할 때도 매핑 테이블을 작성하여 관리
- 가상 주소는, 실제로 물리 주소나 스왑 영역 중 한곳에 위치하며 메모리 관리자는 가상 주소와 물리 주소를 일대일 매핑 테이블로 관리 함
- 가상 주소에 적힌 프로세스에 어떤 값이 필요 할 때는, 매핑 테이블에 적힌 물리 메모리의 위치 데이터에서 데이터를 가져오면 됨

## Paging

### 페이징 기법의 구현

- 고정 분할 방식을 활용한 가상 메모리 관리 기법
- 물리 주소 공간을 같은 크기로 나누어 사용
- 가상 주소는 프로세스 입장에서 바라본 메모리의 공간이기 때문에, 항상 0번지 부터 시작
- 가상 주소의 분할된 각 영역을 페이지라고 부름 ( 페이지 0 / 1 / 2 ... )
- 물리 메모리의 분할된 각 영역은 프레임이라고 부름 ( 프레임 0 / 1 / 2 ... )
- 페이지와 프레임의 크기는 항상 같게 나누기 때문에, 페이지에는 어떤 프레임도 배치 될 수 있음
- 페이지와 프레임이 연결 된 형태는 페이지 테이블 ( 매핑 테이블 )에 숫자의 형태로 담아 놓음
- 페이지에 있는 메모리 구간이, 물리 메모리와 연결 되지 않았을 때는 페이지 테이블에 invalid를 기입 해 놓는데, 이는 swap 영역에 있음을 의미

### 페이징 기법의 주소 변환

![페이지 테이블](/assets/images/posts/2021-03-08-virtual-memory/pic2.png)

- 페이지 테이블은 위 표와 같고, 각 페이지 당 크기는 10 Bit라고 가정
- 가상 주소 38번지에 있는 값을 가져오는 순서
    1. 가상주소 38번지가 어느 페이지에 있는지 확인 → 페이지 3의 8번쨰에 있음
    2. 페이지 테이블의 페이지 3으로 가서 프레임 1에 있는것 확인
    3. 프레임 1의 8번째 위치에 접근하여 값 가져 옴
- 가상주소 12번지에 값 저장하는 순서
    1. 가상주소 12번지가 어느 페이지에 있는지 확인 → 페이지 1의 2번째에 있음
    2. 페이지 테이블의 페이지 1로 가서 프레임 3에 있는것 확인
    3. 프레임 3의 2번째 위치에 접근하여 값 저장 

### 정형화된 주소 변환

- 가상 주소 변환 식
    - VA = <P, D> ( VA → Virtual Address / P → Page / D → Distance ( 페이지의 처음 위치에서 해당 주소까지 거리 ))로 주로 사용
    - 위 경우, 가상주소 38번지는 VA = <3,0> 으로 작성 가능 ( 페이지 3의 8번쨰 주소 )
- 물리 주소 변환 식
    - PA = <F, D> ( PA → Physical Address / F → Frame / D → Distance )로 주로 사용
    - VA = <3, 0>이 PA = <1, 0> 으로 변환 되었다는 것은, 가상주소 30번지가 물리주소 프레임 1의 0번 위치로 변환 되었다는 것을 의미
- 페이징 기법의 주소 변환 과정 : VA = <P, D> → PA = <F, D>
- D는 페이지와 프레임의 크기를 똑같이 나누었기 때문에 일정 함
- 10 Bit로 페이지를 나누면, 사람은 계산하기 편리하겠지만 실제로 컴퓨터는 2진수를 사용하기 때문에 2의 지수승 만큼의 크기로 메모리를 분할한다.
- 이렇게 페이지의 크기가 다양 할 경우 가상 주소를 <P, D>로 변환하는 공식은 아래와 같다.
    - P = 가상 주소 / 한 페이지의 크기 의 몫
    - D = 가상 주소 / 한 페이지의 크기 의 나머지 값
    - 예제
        - 한 페이지의 크기가 10B 가상 메모리 시스템에서, 가상 주소 32번지의 P는 32 / 10의 몫인 3, D는 나머지인 2가 된다.
        - 한 페이지의 크기가 512B인 가상 메모리 시스템에서, 가상 주소의 2049번지의 P는 2049/512의 몫인 4, D는 나머지인 1이다.
        - 16Bit CPU의 컴퓨터에서, 한 페이지의 크기가 2의 10승 인 경우를 가정
            - 이때, 페이지 안의 내용을 표기하기 위해 10Bit가 사용 되고, 6Bit가 남게 된다.
            - 6Bit는 페이지의 번호를 매기는 데 사용된다.
            - 따라서, 1024(2의 10승)개의 주소를 가진 페이지가, 64(2의 6승)개의 페이지로 분할 된다.
            - 물리 주소도 이와 마찬가지로 1024Bit로 나눈다.
- 이 때, 물리 메모리가 가상 메모리 보다 부족 한 경우 스왑 영역을 활용한다.

### 페이지 테이블 관리

- 시스템에 여러 개의 프로세스가 존재하고, 프로세스마다 페이지 테이블이 하나씩 있기 때문에 관리가 힘들다
- 페이지 테이블은 메모리 관리자가 자주 사용하는 자료 구조 이므로, 필요 시 빨리 접근 할 수 있어야 하기 때문에, OS의 한 부분에 페이지 테이블을 모아 놓는다.
- 페이지 테이블이 프로세스마다 존재하는데, 문제는 페이지 테이블의 크기가 작지 않다는 것
    - 32Bit CPU에, 한 페이지가 512B로 페이지를 나눈 상황을 가정
    - 32Bit CPU → 4GB가 물리 메모리의 최대 크기
    - 페이지 테이블 하나의 최대 크기는 24.11MB, 프로세스 40개라면 페이지 테이블만 1GB 차지 할 수도 있음
- 따라서 페이지 테이블의 크기를 적정하게 유지하는 것은 페이지 관리의 핵심
- PCB ( Process Control Block )안에, ( 페이지 테이블 기준 레지스터 )Page Table Base Register 에 페이지 테이블의 시작 주소를 저장 해 놓음
    - 메모리 관리자가 페이지 테이블에 빠르게 접근하기 위함
- 프로세스가 스왑 영역으로 넘어 갈 때, 페이지 테이블의 일부도 스왑 영역으로 넘어가는 경우도 있음

### 페이지 테이블 매핑 방식

- 메모리가 부족 할 때, 페이지 테이블의 일부도 스왑 영역으로 넘어 가는 경우, 페이지 테이블을 참조하는 공식이 변형 됨 ( 페이지 매핑 방식이 변형 됨 )
- 직접 매핑
    - 페이지 테이블 전체가 물리 메모리의 운영체제 영역에 존재하는 방식
    - 별다른 부가 작업 없이 주소 변환 가능
    - 페이지 테이블 전체가 메모리에 올라 와 있기 때문에, 프레임 번호만 적어 줘도 변환 가능 함
    - 페이지 번호만 알면 바로 조회 가능
    - 페이지 테이블 기준 레지스터가 페이지 테이블의 시작 주소를 알 고 있으므로, 이를 기반으로 페이지의 실제 주소를 아는것이 쉬움
- 연관 매핑
    - 페이지 테이블 전체를 스왑 영역에서 관리하는 방식
    - 물리 메모리의 여유 공간이 적을 때 사용하는 방식
    - 페이지 테이블의 일부만 물리 메모리에 가지고 있음 ( 무작위 선택 )
    - 페이지의 일부만 메모리에 있기 때문에, 프레임 번호와 페이지 번호를 둘 다 페이지 테이블에 기입 해야 함
    - 페이지 번호를 알아도 테이블에서 모든 번호를 조회 해야 함, 심지어 메모리에 올라온 테이블 속에 페이지가 없는 경우 스왑 영역에 가서 조사 해야 함
    - 일부 페이지 테이블이 물리 메모리에 올라 와 있는 것을 변환 색인 버퍼 ( Translation Look-aside Buffer )라 부름
    - 캐시 시스템과 유사
        - 원하는 데이터가 캐시에 있는 경우 캐시 히트, 없는 경우 캐시 미스인데 이 경우도 마찬가지임
    - 원하는 페이지 번호가 변환 색인 버퍼에 있는 경우엔, TLB Hit / 없는 경우엔 TLB Miss 라고 칭함
    - TLB Miss가 자주 발생할 경우 시스템의 성능이 저하 됨
- 집합 연관 매칭
    - 연관 매핑의 문제점을 개선 한 방식
    - 모든 페이지 테이블을 스왑 영역에서 관리 및 일부만 메모리로 가져오는 점은 동일
    - 페이지 테이블을 일정한 집합으로 자르고, 자른 덩어리 단위로 물리 메모리에 가져 옴
    - 페이지 테이블을 n개씩 자르고, 이를 관리하는 페이지 테이블( 디렉터리 테이블 )을 하나 더 생성 할 수 있음
    - 관리 페이지 테이블에는, 스왑 영역에 있는 페이지 테이블과, 물리 메모리에 있는 페이지 테이블을 표기 함
    - 따라서 스왑 영역에 있거나, 물리 영역에 있는 것을 연관 매핑처럼 일일이 검색하지는 않아도 됨
    - 페이지 테이블이 일정 크기의 묶음으로 나뉘기 때문에, 가상주소를 VA = <P, D> 가아니라, VA = <P1, P2, D>로 바꾸어 표기 ( P1 → 디렉터리 테이블에서의 위치 정보, P2 → 묶음 내에서의 위치 정보 )
    - P1을 통해 P2 테이블 덩어리가 메모리 영역에 있는지, 스왑 영역에 있는지 확인 할 수 있다
- 역매핑
    - 페이지 번호를 기준으로 테이블을 구성하는 위 세가지 방식과 달리, 물리 메모리의 프레임 번호를 기준으로 테이블을 구성
    - PID ( Process ID )를 기준으로 테이블을 구성하며, PID가 어떤 페이지에 올라 와 있는지를 표기 한 테이블
    - 프로세스 수와 상관 없이 테이블이 하나만 존재 → 크기가 매우 작음
    - 가상메모리에 접근 할 때 PID / 페이지 번호를 모두 찾아야 하는 단점이 있음
    - 프로세스 1의 페이지 10이 메모리에 있는지 확인하려면, 페이지 테이블에 있는 모든 값과 대조를 해 보아야만 알 수 있음
    - 페이지 테이블의 행 수는, 실제 프레임의 수와 같아 프로세스 수와 상관 없이 항상 일정 크기의 페이지 테이블을 유지하여 테이블의 크기가 매우 작다.

## Segmentation

### 세그먼테이션 기법의 구현

- 물리 메모리를 프로세스의 크기에 따라 가변적으로 나누어 사용
- 페이징 기법과 마찬가지로 세그먼테이션 테이블을 사용
- segment의 크기를 나타내는 limit / 메모리상의 시작 주소를 나타내는 address가 테이블에 기입 되어 있음
- 각 segment가 주어진 메모리 영역을 넘어가면 안되기 때문에 segment의 크기 정보에는 size 대신 limit 사용
- 물리 메모리 부족 시 swap영역 사용하는데, address에 I ( invalid )값을 대입하여 사용
- 가변 분할 방식과 동일하므로, 장단점 또한 동일

### 세스먼테이션 기법의 주소 변환

- VA = <S, D>로 표현 ( S → Segment number / D → 세그먼트 시작 지점에서 해당 주소까지의 거리 ( Distance )
- 예시

    ![Segmentation table](/assets/images/posts/2021-03-08-virtual-memory/pic3.png)

    - 위와 같은 segmentation table와, 첫번째 프로세스의 32번째 주소에 접근 하는 경우를 가정
    - 첫번째 프로세스는 segment 0으로 분할되었으므로, S는 0이고, D는 32이므로 VA = <0, 32> 이다.
    - segment 0의 시작 주소를 위 테이블에서 보면 120이므로, 120에 32를 더하여 물리주소 152를 구함
    - 이때, 메모리 관리자는 거리가 segment의 크기보다 큰 지를 항상 검사
    - segment의 크기보다 거리가 크지 않다면, 물리주소 152에 접근하여 원하는 데이터를 읽거나 씀

## Segmentation - Paging 혼용

### 메모리 접근 권한

- 메모리의 특정 번지에 저장 된 데이터를 사용 할 수 있는 권한
- 읽기 / 쓰기 / 실행 / 추가의 권한이 있음
- 하지만 추가 권한의 경우, 쓰기와 항상 같이 취급 되어야 하므로, 읽기 / 쓰기 / 실행 세가지로 권한을 관리 → 총 8가지의 경우가 나옴 ( 모두 허용 ~ 모두 비허용 )
- 일반적으로, 프로세스의 코드 영역은 자기 자신을 수정하는 프로그램은 없기 때문에, 읽기 및 실행 권한을 가짐
- 데이터 영역은 읽고 / 쓰기가 가능한 영역 또는, 읽기만 가능한 영역으로 나눌 수 있음 → 상수로 선언한 부분은 읽기만 가능
- 이 메모리 접근 권한 검사는 가상 주소에서 물리 주소로 주소 변환이 일어 날 때마다 시행 됨, 읽기만 되는 곳에 쓰기를 하려고 하면 Trap 메모리 오류 발생

### Segmentation - paging 혼용 기법 도입

- 페이지 마다 접근 권한이 다르기 때문에, 페이지 테이블의 모든 행에는 메모리 접근 권한과 관련된 권한 비트 ( right bit )가 추가 됨
- 권한 비트를 기반으로 알맞은 접근인지 아닌지 매번 판단
- 권한 비트가 추가됨으로써 페이지 테이블의 크기가 커짐
- segmentation table을 활용하여 권한 비트 문제를 해결 할 수 있음
- 권한 비트가 같은 것 끼리 segment로 나누고, 나누어진 결과물을 segmentation table에 저장
- 각 페이징 단위는 유지
- 페이징 기법과 segmentation 기법의 장점만 합쳐 놓은 방식으로, 많은 OS가 이 방법을 사용 중

### Segmentation - paging 주소 변환

- VA = <S, P, D>로 표현 ( S → Segmentation number / P → Page number / D → Distance )
- 페이징 기법의 가상 주소에 segment번호 S가 추가 된 것
- 순서
    1. 사용자가 어떤 주소에 있는 데이터를 요청 하면, 해당 주소가 몇 번째 segment의 몇 번째 페이지로부터 얼마나 떨어져있는지 게산하여 가상 주소 VA = <S, P, D>를 구함
    2. S에 해당하는 segment 번호로 가서, 불법 접근이 아닌지 등을 확인 
    3. 2에서 trap이 발생하지 않으면, 연결된 페이지로 가서 어느 프레임에 저장되었는지 찾음 ( 있다면, 메모리에 바로 접근하고, 없다면 스왑 영역에 가서 해당 페이지를 물리 메모리로 가져 옴 )
    4. 물리 메모리에 있는 프레임의 처음 위치에서, D만큼 떨어 진 곳에 접근하여 데이터를 읽거나 씀