---
title:  "1장 안전성 item 7"
excerpt: "결과 부족이 발생할 경우 null과 Failure를 사용하라."

categories:
  - effective_kotlin
tags:
  - [kotlin]

toc: true
toc_sticky: true
 
date: 2024-03-29
last_modified_at: 2024-03-29
---

### 1장 안정성_결과 부족이 발생할 경우 null과 Failure를 사용하라

함수가 원하는 결과를 만들 수 없을 때가 있음

- 서버로 부터 데이터를 읽어들이려고 했는데 인터넷 문제로 읽지 못한 경우
- 조건에 맞는 첫번째 요소를 찾으려고 했는데, 조건에 맞는 요소가 없는 경우
- 텍스트를 파싱해서 객체를 만들려고 했는데, 텍스트의 형식이 맞지 않는 경우

이러한 상황을 처리하는 메커니즘은 크게 두 가지가 있음

- null 또는 ‘실패를 나타내는 sealed 클래스(Failure라고 함 보통)
- 예외를 throw

위 두가지는 중요한 차이점이 있음.

예외는 정보를 전달하는 방법으로 사용하면 안됨. 잘못된 특별한 상황을 나타내야 하며, 처리되어야함.

1. 많은 개발자가 예외가 전파되는 과정을 제대로 추적하지 못함
2. 코틀린의 모든 예외는 unchecked 예외.
3. 예외는 예외적인 상황을 처리하기 위해 만들어졌으므로, 명시적인 테스트만크 빠르게 동작하지않음
4. try-catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한됨.

반면, null과 Failure 는 예상되는 오류를 표현할 때 굉장히 좋음.

따라서, 충분히 예측할 수 있는 범위의 오류는 null과 Failure를 사용하고, 예측하기 어려운 예외적인 범위의 오류는 예외를 throw해서 처리하는 것이 좋음.

```kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
    // ...
    if (incorrectSign) {
        return null
    }
    // ...
    return result
}

inline fun <reified T> String.readObject(): Result<T> {
    // ...
    if (incorrectSign) {
        return Failure(JsonParsingException())
    }
    // ...
    return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()

class JsonParsingException: Exception()
```

이렇게 표시되는 오류는 다루기 쉬움. null을 처리해야 한다면, 사용자는 안전 호출 또는 Elvis 연산자 같은 다양한 null-safety 기능을 활용

```kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```

Result와 같은 공용체를 리턴하기로 했다면, when 표현식을 사용해서 이를 처리할 수 있음.

```kotlin
val person = userText.readObject<Person>()
val age = when(person) {
    is Success -> person.age
    is Failure -> -1
}
```

이러한 오류 처리 방식은 try-catch 블록보다 효율적이며, 더 쉽고 명확함.

예외는 놓칠 수도 있고, 애플리케이션을 중지 시킬 수도 있음. null값과 sealed result 클래스는 명시적으로 처리해야 함. null값과 sealed result 클래스는 명시적으로 처리해야함.

**null값과 sealed result 클래스의 차이**

- 추가적인 정보를 전달해야 한다면 = sealed result 클래스
- 그렇지 않으면  = null

일반적으로 두 가지 형태의 함수를 사용. 하나는 예상할 수 있을 때, 하나는 없을 때.

- get : 특정 위치에 있는 요소를 추출. 만약 없으면 IndexOutBoundsException을 발생
- getOrNull : out of range 오류가 발생하는 경우, null을 리턴

nullable을 리턴하지 말아야 한다. 개발자에게 null이 발생 할 수 있다는 경고를 주려면, getOrNull등을 사용해서 무엇이 리턴 되는지 예측할 수 있게 하는 것이 좋다.