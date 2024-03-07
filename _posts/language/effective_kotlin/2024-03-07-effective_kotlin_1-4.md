---
title:  "1장 안전성 item 4"
excerpt: "inferred 타입으로 리턴하지 말라. "

categories:
  - effective_kotlin
tags:
  - [kotlin, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2024-03-07
last_modified_at: 2024-03-07
---

### item4. inferred 타입으로 리턴하지 말라

코틀린의 타입 추론은 JVM 세계에서 가장 널리 알려진 코틀린의 특징.  몇가지 위험한 부분은 있음.

```kotlin
open class Animal
class Zebra: Animal() 

fun main() {
    var animal = Zebra() //타입 추론을 하여 Zebra 타입으로 초기화
    animal = Animal()  //상위 클래스인 Animal로 초기화 X
}
```

타입을 명시적으로 지정하면 이러한 무제를 해결 할 수 있음

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
    var animal: Animal = Zebra()
    animal = Animal()
}
```

직접 라이브러리 또는 모듈을 조작할 수 없는 경우 이러한 문제를 간단히 해결할 수 없음. 이러한 경우 inferred 타입을 노출하면, 위험한 일이 발생할 수 있음.

```kotlin
interface CarFactory {
    fun produce(): Car
}
```

위는 자동차를 생성하는 인터페이스

아래는 디폴트로 생성되는 자동차가 있다고 가정

```kotlin
val DEFAULT_CAR: Car = Fiat126p()
```

대부분의 공장에서는 Fiat126p를 생성하기 때문에 디폴트로 두었음. Car가 명시적으로 지정되어있으므로 리턴타입을 제거했다고 가정

```kotlin
interface CarFactory {
    fun produce() = DEFAULT_CAR
}
```

그러다가 타입추론에 의해 타입이 자동으로 지정될 것 이라고 생각하여 다음과 같이 변경.

```kotlin
val DEFAULT_CAR = Fiat126P()
```

이제 Fiat126P 밖에 못만든다.

만약 인터페이스를 직접 만들었다면, 문제를 굉장히 쉽게 찾아서 수정할 수 있었을 것. 하지만 외부 API라면 문제를 쉽게 해결할 수 없을 것.

### 정리

타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정.