---
title:  "1장 안전성 item 3"
excerpt: "최대한 플랫폼 타입을 사용하지 말라. "

categories:
  - effective_kotlin
tags:
  - [kotlin, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2024-03-07
last_modified_at: 2024-03-07
---

### item3. 최대한 플랫폼 타입을 사용하지 말라

null-safety는 코틀린의 주요 기능 중 하나.

```kotlin
//자바
public class JavaTest {

    public  String giveName() {
        // ...
    }
}
```

위와 같이 String 타입의 리턴값에 @NotNull 어노테이션이 붙어있지 않다면 nullable 이라고 가정할 수 있다.

```kotlin
//자바
public class UserRepo {
    public  List<User> getUsers() { //자바에서는 모든것이 nullable일 수 있음
        // ...
    }
}

//코틀린
val users: List<User> = UserRepo().users!!.filterNotNull() //null 이 아닐꺼라는 확신
```

코틀린이 디폴트로 모든 타입을 nullable로 다룬다면, 우리는 이를 사용할 때 이러한 리스트와 리스트 내부의 User 객체들이 널이 아니라는것을 알아야 한다. 따라서 리스트 자체만 널인지 확인해서는 안되고 그 내부에 있는 것들도 널인지 확인해야함.

리스트는 적어도 map 과 filterNotNull 등의 메서드를 제공. 하지만 다른 제네릭 타입이라면, 널을 확인하는 것 자체가 복잡. 그래서 코틀린은 자바 등, 다른 프로그래밍 언어에서 넘어온 타입들을 특수하게 다룸. 이러한 타입을 **플랫폼 타입(platform type)** 이라고 함.

플랫폼 타입은 String! 처럼 타입 이름 뒤에 ! 기호를 붙여서 표기. 물론 이러한 노테이션이 직접적으로 코드에 나타나지는 않음. 다음과 같은 형태로 이를 선택적으로 사용

```kotlin
// 자바
public class UserRepo {
    public User getUsers() {
        // ...
    }
}

//코틀린
val repo = UserRepo()
val user1 = repo.user           //user1의 타입은 User!
val user2: User = repo.user     //user2의 타입은 User
val user3: User? = repo.users   //user3의 타입은 User?
```

문제는 null이 아니라고 생각했던 필드가 null일 가능성이 있으므로, 여전히 위험함. 그래서 플랫폼 타입을 사용할 때는 항상 주의를 기울여야 한다.

자바를 코틀린과 함께 사용할 때에는 자바 코드를 직접 조작할 수 있는 상황이라면 가능한 한 @Nullable, @NotNull 어노테이션을 붙여서 사용하는 것이 좋다.

```kotlin
public class UserRepo {
    public  @NotNull User getUsers() {
        // ...
    }
}
```

### 정리

다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 플랫폼 타입이라고 함. 이런 코드는 빨리 제거하는 것이 좋음. 또한 연결되어있는 자바 생성자, 메서드, 필드에 nullabale 여부를 지정하는 어노테이션을 활용하는 것도 좋음.