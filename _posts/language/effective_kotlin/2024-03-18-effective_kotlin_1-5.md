---
title:  "1장 안전성 item 5"
excerpt: "예외를 활용해 코드에 제한을 걸어라."

categories:
  - effective_kotlin
tags:
  - [kotlin, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2024-03-18
last_modified_at: 2024-03-18
---

### 1장 안정성_예외를 활용해 코드에 제한을걸어라

확실하게 어떤 형태로 동작해야 하는 코드가 있다면, 예외를 활용해 제한을 걸어주는 것이 좋다.

- require 블록 : 아규먼트를 제한할 수 있음.
- check 블록 : 상태와 관련된 동작을 제한할 수 있음.
- assert 블록 : 어떤것이 true인지 확인할 수 있음. 테스트 모드에서만 작동.
- return 또는 throw와 함께 사용하는 Elvis 연산자

```kotlin
fun pop(num: Int = 1): List<T> {
    require(num <= size ){ //참이 아니면 예외를 발생
        "Cannot remove more elements than current size"
    }
    check(isOpen) {"Cannot pop from closed stack"}  //false 일 경우 예외를 발생
    val ret = collection.take(num)
    collection = collection.drop(num)
    assert(ret.size == num)  //테스트모드에서 false인겨우 예외를 발생
    retrun ret
}
```

제한을 걸었을 때 장점

- 문서를 읽지 않은 개발자도 문제를 확인할 수 있음.
- 문제가 있는 경우, 예상하지 못한 동작을 하지 않고 예외를 throw. 이러한 제한으로 문제를 놓치지 않고 코드가 더 안정적으로 동작.
- 스마트 캐스트 기능을 활용할 수 있게 되므로, 타입 변환을 적게 할 수 있음.

### 아규먼트

함수를 정의할 때 타입 시스템을 활용해서 argument에 제한을 거는 코드를 많이 사용함.

- 숫자를 아규먼트로 받아서 팩토리얼을 계산한다면 숫자는 양의 정수여야함.
- 좌표들을 아규먼트로 받아서 클러스터를 찾을 때는 비어 있지 않은 양의 좌표 목록이 필요
- 사용자로부터 이메일 주소를 입력받을 때는 값이 입력되어 잇는지, 이메일 형식이 맞는지 확인해야함.

일반적으로 이러한 제한을 걸 때는 require 함수를 사용. require 함수는 제한을 확인하고, 제한을 만족하지 못할 경우 예외를 throw

```kotlin
fun factorial(n: Int): Long {
    require(n >= 0 )
    return if (n <= 1) 1 else factorial(n-1) * n
}
fun findCluster(points: List<Point>): List<Cluster> {
    require(points.isNotEmpty())
    //..
}

fun sendEmail(user: User, message: String) {
    requireNotNull(user.email)
    require(isValidEmail(user.email))
    //..
}
```

이와 같은 형태의 입력 유효성 검사 코드는 함수의 앞부분에 배치.

람다를 활용하여 지연 메세지를 정의할 수 있음.

```kotlin
fun factorial(n: Int): Long {
    require(n >= 0) { //람다를 인자값으로 받음
    "Cannot calculate factorial of $n because it smaller than 0"
    }
    return if (n <= 1) 1 else factorial(n-1)*n
}
```

### 상태

어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야 할 때가 있음

- 어떤 객체가 미리 초기화되어 있어야만 처리를 하게 하고싶은 함수
- 사용자가 로그인 했을 때만 처리를 하고 싶은 함수
- 객체를 사용할 수 있는 시점에 사용하고 싶은 함수

상태와 관련된 제한을 걸때는 일반적으로 check 함수를 사용

```kotlin
fun speak(text: String){
    check(isInitialzed)
    //...
}

fun getUserInfo(): UserInfo {
    checkNotNull((token))
}

fun next():T {
    check(isOpen)
    //...
}
```

check 함수는 require와 비슷하지만, 지정된 예측을 만족하지 못할 때, IllegalStateException을 throw함. 상태가 올바른지 확인. 일반적으로 require 뒤에 배치.

### Assert 계열 함수 사용

함수가 올바르게 구현되어 있다는 것을 확인하려면 단위 테스트를 하는 것이 좋다.

### nullability와 스마트 캐스팅

코틀린에서 require와 check 블록으로 어떤 조건을 확인해서 true가 나왔다면, 해당 조건은 이후로도 true일거라고 가정

```kotlin
public inline fun require(value: Boolean): Unit {
    contract { //함수의 사전조건, 사후조건 및 불변식을 명시적으로 선언
        returns() implies value //만약 함수가 반환된다면
    }
    require(value) { "Failed requirement."}
}
```

###