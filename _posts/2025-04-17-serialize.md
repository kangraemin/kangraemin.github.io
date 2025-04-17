---
title: Android - 직렬화      
date: 2025-04-17
categories:
 - Android
tags:
 - Android
 - Serialization
 - parcelable
 - parcelize 
---

Android 에서 주로 쓰이는 직렬화 방식에 대해 알아봅니다.   

<!-- more -->

### 직렬화

<aside>
💡객체의 상태를 ByteStream 으로 변환하여 저장 / 전달하는 기법
</aside>

- 메모리에 있는 객체를 파일 / 네트워크 / 프로세스간 통신 ( IPC ) 에서 사용 할 수 있는 형태로 변경
- Android 에서는 주로 Java의 Serializable 인터페이스, Android의 Parcelable 인터페이스 활용
- 직렬화 ( Serialization )
  - 객체의 상태 ( 변수 값 등 )을 연속적인 바이트 형태로 변환하는 과정
- 역직렬화 ( Deserialization )
  - 저장된 바이트 스트림에서 다시 객체를 복원하는 과정

### Java Serializable

<aside>
💡객체를 스트림으로 변환하여 저장 / 전송 할 수 있도록 하는 java marker interface
</aside>

- java.io.Serializable 인터페이스 구현
  - 해당 인터페이스는, marker 역할만 하며, 별도의 메서드를 강제하지 않음
- Reflection 기반으로, 객체의 필드들을 순회하며 바이트 스트림으로 변환
  - 리플렉션 사용에 따른 성능 저하가 발생 할 수 있음
- writeObject / readObject 를 선언 해 둔다면, 해당 함수를 통해 직렬화 커스텀 가능
- 필요한 필드만 직렬화를 하고 싶은 경우, transient 키워드 활용 가능
- 주의점
  - 복잡한 객체의 경우 성능 부담이 있음
  - 클래스 구조가 변경 될 때, serialVersionUID 를 정의하지 않으면 역직렬화시 문제가 발생 할 수 있음

### Java Parcelable

<aside>
💡객체를 스트림으로 변환하여 저장 / 전송 할 수 있도록 하는 Android interface
</aside>

- Java Serializable 처럼, 직렬화 / 역직렬화를 위한 interface
- Java Serializable에 비해 성능이 뛰어나고, 메모리 할당이 효율적이지만 직접 구현해야 하므로 보일러 플레이트 코드가 다소 많음
- Parcelable 인터페이스를 구현하면, 데이터를 Parcel 이라는 바이트 버퍼에 저장하여 Intent / Bundle / IPC 를 통해 데이터 전달 가능
- 각 필드를 Parcel 에 기록하고, 동일한 순서로 읽어드림 → 순서가 맞지 않으면 정상적인 복원이 안될수도
- 메서드
  - `writeToParcel(Parcel dest, int flags)` 객체의 필드를 dest 에 기록
  - `describeContents()` 파일 디스크립터와 관련된 특별한 내용이 있다면 반환하고, 일반적으로 0 반환
  - `CREATOR` Parcel 에서 객체를 생성 할 수 있도록 도와주는 팩토리 생성
  - 예시 코드

      ```kotlin
      import android.os.Parcel;
      import android.os.Parcelable;
      
      public class Person implements Parcelable {
          private String name;
          private int age;
      
          // 일반 생성자
          public Person(String name, int age) {
              this.name = name;
              this.age = age;
          }
      
          // Parcel을 통해 객체를 생성하는 생성자
          protected Person(Parcel in) {
              // 객체의 필드를 쓰여진 순서와 동일하게 읽어야 함
              name = in.readString();
              age = in.readInt();
          }
      
          // writeToParcel: 객체 데이터를 Parcel에 기록
          @Override
          public void writeToParcel(Parcel dest, int flags) {
              // 필드를 동일한 순서로 기록
              dest.writeString(name);
              dest.writeInt(age);
          }
      
          // describeContents: 보통 0 리턴 (특별한 경우 FILE_DESCRIPTOR 등이 있을 경우 다른 값)
          @Override
          public int describeContents() {
              return 0;
          }
      
          // CREATOR: Parcelable 객체를 생성하기 위한 팩토리 역할
          public static final Creator<Person> CREATOR = new Creator<Person>() {
              @Override
              public Person createFromParcel(Parcel in) {
                  return new Person(in);
              }
      
              @Override
              public Person[] newArray(int size) {
                  return new Person[size];
              }
          };
      
          @Override
          public String toString() {
              return "Person{name='" + name + "', age=" + age + "}";
          }
      
          // Getter/Setter 생략 가능 (필요 시 추가)
      }
      ```


### Kotlin Parcelize

<aside>
💡코틀린 플러그인을 통해, 보일러플레이트 코드를 제거하고 Parcelable 을 구현 할 수 있게 만듬
</aside>

- 플러그인 필요

    ```kotlin
    plugins {
        id 'com.android.application'
        id 'kotlin-android'
        id 'kotlin-parcelize'  // 이 플러그인을 추가합니다.
    }
    ```

- @Pacelize 어노테이션을 활용
  - Kotlin 컴파일러 플러그인이 자동으로 `writeToParcel()` / `describeContents()` / `CREATOR` 필드를 생성 해 줌
  - 데이터 클래스를 활용하면, 객체의 상태 ( 프로퍼티 ) 가 자동으로 처리 됨
  - 기본 자료형 뿐 아니라, List / Map 등도 Parcelable 처리 가능
  - 예시 코드

      ```kotlin
      import kotlinx.parcelize.Parcelize
      import android.os.Parcelable
      
      @Parcelize
      data class Person(
          val name: String,
          val age: Int,
          val hobbies: List<String>  // List와 같은 컬렉션도 Parcelable 조건을 만족하면 사용 가능
      ) : Parcelable
      ```