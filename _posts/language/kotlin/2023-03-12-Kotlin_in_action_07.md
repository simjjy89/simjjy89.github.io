---
title:  "코틀린 인 액션 07장 연산자 오버로딩과 기타 관례"
excerpt: "연산자오버로딩, 관례, 위임 프로퍼티에 대해 알아보자"

categories:
  - kotlin
tags:
  - [kotlin, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2023-03-12
last_modified_at: 2023-03-12
---
## 연산자 오버로딩과 기타 관례
코틀린에서는 어떤 언어 기능이 특정타이비(클래스)와 연관 되기 보다는 특정 함수 이름과 연관된 경우가 있음. 이런식으로 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법을 관례(convention) 이라고 함.

### 산술 연산자 오버로딩
- 자바의 산술 연산자 
  - 에서는 원시타입에만 산술 연산자를 사용할 수 있고, String 타입에 대해 + 연산자를 사용할 수 있다.
- 코틀린의 산술연산자
  - 원시타입 뿐만 아니라 다른 클래스에서도 산술연산자를 사용하여 직관적인 표현이 가능
  ```kotlin
        val num1: BigInteger = BigInteger("100")
        val num2: BigInteger = BigInteger("200")
        
        // 동일하게 동작한다.
        val num3 = num1.add(num2)  
        val num3 = num1 + num2
  ```

### 이항 산술 연산 오버로딩
```kotlin
data class Point(
    val x: Int, val y: Int
){
    //멤버 함수로 선언
    operator fun plus(other: Point): Point { //"plus" 연산자 함수를 정의!
        return Point(x = x+other.x, y=y+other.y)
    }
}

//멤버 함수 대신 확장 함수로 정의 할 수도 있음
operator fun Point.plus(other: Point): Point {
  return Point(x + other.x, y + other.y)
}

  
val p1 = Point(10,20)
val p2 = Point(30,40)
println(p1+p2) //컴파일 될때 p1.plus(p2)로 됨

------------------------------------------------------------
Point(x=40, y=60)
```
- 연산자를 오버로딩하는 함수 앞에는 꼭 operator 키워드가 있어야 함. 관례를 따르는 함수임을 나타내는 키워드!!
- 멤버함수대신 확장 함수로 연산자 함수를 정의할수도 있음
- 직접 연산자를 만들어 사용할 수 없고 미리 정해둔 연산자만 오버로딩 할 수 있음.

|식|함수 이름|
|------|---|
|a * b|times|
|a / b|div|
|a % b|mod(1.1 부터 rem)|
|a + b|plus|
|a - b|minus|

- 연사자를 정의할 때 두 피연사자의 타입이 같을 필요는 없음
```kotlin
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}

val p1 = Point(10, 20)  
println(p1 * 1.5)

------------------------------------------------------------
Point(x=15, y=30)
```

- 연잔자 함수 반환 타입이 피연산자 중 하나와 일치할 필요도 없음
```kotlin
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}

print('A' * 10)
------------------------------------------------------------
AAAAAAAAAA
```


