---
title:  "1장 안전성 item 2"
excerpt: "변수의 스코프를 최소화하라. "

categories:
  - effective_kotlin
tags:
  - [kotlin, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2024-03-05
last_modified_at: 2024-03-05
---

### item2. 변수의 스코프를 최소화하라

상태를 정의할 때는 프로퍼티의 스코프를 최소화하는것이 좋다.

- 프로퍼티보다는 지역변수를 사용
- 최대한 좁은 스코프를 갖게 변수를 사용. 반복문 내부에서만 변수가 사용된다면, 변수를 반복문 내부에 작성

```kotlin
val a = 1
fun fizz() {
    val b =2
    println(a+b)
}

val buzz = {
    val c = 3
    println(a + c)
}
// 이 위치에서 a를 사용할 수 있지만, b와 c는 사용할 수 없다.
```

fizz와 buzz는 외부 스코프에 있는 변수에 접근할 수 있지만 외부 스코프에서는 내부 스코프에 정의된 변수에 접근할 수 없음.

```kotlin
data class User(
    val name: String
)

fun main(){
    val users = listOf(User("심"),User("박"))

    //나쁜 예
    var user1: User
    for (i in users.indices){
        user1 = users[i]
        println("User at $i is $user1")
    }

    //조금 더 좋은 예
    for (i in users.indices){
        val user2= users[i]
        println("User at $i is $user2")
    }

    //제일 좋은 예
    for((i, user3) in users.withIndex()){
        println("User at $i is $user3")
    }
}
```

스코프를 좁게 만드는 가장 중요한 이유는 프로그램을 추적하고 관리하기 쉽기 때문.

mutable 프로퍼티는 좁은 스코프에 걸쳐있을수록, 그 변경을 추적하는 것이 쉬음.

변수는 정의할 때 초기화 되는 것이 좋음. if, when, try-catch, Elvis 표현식 등을 활용하면, 최대한 변수를 정의할 때 초기화할 수 있습니다.

```kotlin
    //나쁜예
    val user: User
    if(hasValue) {
        user = getValue()
    } else {
        user = User()
    }

    //조금 더 좋은 예
    val user: User = if(hasValue){
        getValue()
    } else {
        User()
    }
```

여러 프로퍼티를 한꺼번에 설정해야 하는 경우에는 구조분해 선언(destructuring declaration)을 활용하는 것이 좋다.

```kotlin
//나쁜예
fun updateWeather(degrees: Int) {
    val description: String
    val color: Int
    if (degrees < 5) {
        description = "cold"
        color = Color.BLUE.num
    } else if (degrees < 23) {
        description = "cold"
        color = Color.YELLOW.num
    } else {
        description = "cold"
        color = Color.RED.num
    }

    println(color)
}

//조금 더 좋은 예
fun updateWeather(degrees: Int){

    val (description, color) = when {
        degrees < 5 -> "cold" to Color.BLUE.num
        degrees < 23 -> "mil;d" to Color.YELLOW.num
        else -> "hot" to Color.RED
    }

    println(color)
}
```

### 캡처링

이부분은 아직 이해가 잘 안감

```kotlin
    var numbers = (2..100).toList() 
    val primes = mutableListOf<Int>()
    while (numbers.isNotEmpty()) {
        val prime = numbers.first()
        primes.add(prime)
        numbers = numbers.filter { it % prime != 0 }
    }
    println(primes) //[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97]
```
