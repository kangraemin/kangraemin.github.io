---
title: Python 얕은 복사와 immutable 변수, small integer caching
date: 2021-03-19
categories:
 - Python
tags:
 - Python
---

[Facebook Python Korea](https://www.facebook.com/groups/pythonkorea/permalink/3814700018613131/)에 올라온 질문 글에 대해 공부 한 얕은 복사, immutable 변수에 대한 내용을 공유합니다. 

<!-- more -->

### immutable 변수와 얕은 복사 

위 Python Korea에 올라온 질문을 내용은 다음과 같습니다. 

```Python
a = 3
b = a
print(id(a) == id(b)) # True
a += 4
print(id(a) == id(b)) # False
print(a) # 7
print(b) # 3

class A:
    property = 1
c = A()
d = c
print(id(c) == id(d)) # True
print(c.property) # 1
print(d.property) # 1
c.property = 100
print(d.property) # 100
```

위 코드에서, a의 값을 변경 시켰을 때는 b에 영향을 주지 않지만, 클래스 객체인 c의 property 값을 변경 시키면 d의 property 값도 함께 변경되는데 이 이유가 무엇 인지가 질문 내용입니다. 

먼저, python에서 int / str / float 등의 변수 타입은 immutable 타입입니다. 

immutable 한 타입은 `b = a` 형태와 같이 값 대입이 이루어 지면, a와 b는 같은 id값을 가지게 됩니다. 

하지만, 변수의 값이 변하는 순간 변한 변수의 id 값은 변경되게 됩니다. 

따라서, 위 경우 immutable type 변수의 값(a += 4 연산을 했다고 가정)이 변경 되면, 해당 변수(a)와 해당 변수로 초기화된 변수 (b)는 별개의 변수가 됩니다. 

하지만, mutable한 변수의 경우, `b = a` 형태와 같이 값 대입이 이루어 지면, b에 객체 a에 대한 얕은 복사가 이루어 집니다. 

얕은 복사에선, 변수가 같은 메모리 주소값을 계속해서 바라보고 있게 됩니다. 

따라서 a 변수 내부의 property 값이 변경 되는 것이 b 변수 내부의 property 값에도 영향을 미치는 것 입니다. 

따라서 변수 내부의 값이 변하더라도 독립적인 변수를 갖고 싶다면 얕은 복사가 아닌 깊은 복사 (deep copy)가 이루어 져야 합니다. 

아래의 코드를 통해 이와 같은 사실을 확인 할 수 있습니다. 

```Python
>>> a = 12314
# immutable 타입의 변수가 `b = a`형태로 값이 할당 되었을 때 같은 id 값을 가지는 것을 확인 할 수 있습니다. 
>>> b = a
>>> id(a)
140443413147336
>>> id(b)
140443413147336
>>> a += 4
# immutable 타입의 변수의 값이 변하면, id값도 변경 되는것을 확인 할 수 있습니다. 
>>> id(a)
140443413147384
>>> id(b)
140443413147336

>>> class testClass: a = 1
>>> a = testClass()
>>> a.a
1
>>> b = a
>>> b.a
1
# 클래스 객체 a, b가 같은 id 값을 같는 것을 확인 할 수 있습니다. 
>>> id(a)
4412469048
>>> id(b)
4412469048
# 클래스 객체의 property가 변경 되면, 다른 객체에도 영향을 주는것을 확인 할 수 있습니다. 
>>> a.a = 5
>>> id(a)
4412469048
>>> id(b)
4412469048
>>> b.a
5
# compy.deepcopy()를 통해 클래스 객체를 복사합니다.
# 깊은 복사로 초기화된 객체 (c)는 기존의 객체(a,b)와 다른 id값을 가지는 것을 확인 할 수 있습니다. 
>>> import copy
>>> c = copy.deepcopy(a)
>>> id(a)
4412469048
>>> id(b)
4412469048
>>> id(c)
4412469120
# 깊은 복사로 초기화된 객체는 기존의 객체의 property 변경에 영향을 받지 않는것을 확인 할 수 있습니다.
>>> a.a = 123
>>> a.a
123
>>> c.a
5
```