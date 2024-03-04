---
title:  "1장 안전성 item 1"
excerpt: "가변성을 제어하라. "

categories:
  - effective_kotlin
tags:
  - [kotlin, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2024-03-04
last_modified_at: 2024-03-04
---

## item 1. 가변성을 제한하라

코틀린은 모듈로 프로그램을 설계함. 이러한 요소 중 일부는 상태(state)를 가질 수 있음.

```kotlin
var a = 10  
var list: MutableList<Int> = mutableListOf()
```

시간 변화에 따라서 변화하는 요소를 표현하는 것은 유용하지만, 상태를 적절히 관리하는 것은 생각보다 어려움

1. 프로그램 이해와 디버그 하기 힘들어짐. 상태 변경이 많아지면 추적하기 힘들어짐. 이러한 클래스는 이해하기도 어렵고, 수정도 어려움. 예상하지 못한 상황 또는 오류를 발생시키는 경우도 있음.
2. 가변성이 있으면, 코드의 실행을 추론하기 어려움. 시점에 따라 값이 달라질 수 있어서 현재 어떤 값을 가지고 있는지 알아야 코드의 실행을 예측 할 수 있음.
3. 멀티 스레드 프로그램일 때는 적절한 동기화가 필요함. 변경이 일어나는 모든 부분에서 충돌이 발생할 수 있음.
4. 테스트하기가 어렵다. 변경이 많을수로 더 많은 조합을 테스트해야함.
5. 상태 변경이 일어 날 때, 이러한 변경을 다른 부분에 알려야 하는 경우가 있음.

가변성은 시스템의 상태를 나타내기 위한 중요한 방법이지만 변경이 일어나야 하는 부분은 신중하고 확시하게 결정

### 코틀린에서 가변성 제어하기

코틀린은 가변성을 제어할 수 있게 설계되어있음. immutable(불변) 객체를 만들거나, 프로퍼티를 변경할 수 없게 막는것이 굉장히 쉬움.

- 읽기전용 프로프티(val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분하기
- 데이터 클래스의  copy

**읽기전용 프로퍼티**

코틀린은 val 를 이용해 읽기 전용 프로퍼티를 만들 수 있음.

```kotlin
val a = 10
a = 20 // 오류
```

읽기 전용 프로퍼티가 mutable 객체를 담고있다면, 내부적으로 변할 수 있음

```kotlin
val list = mutableListOf(1,2,3)
list.add(4)

print(list) // [1,2,3,4]
```

읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 getter 로도 정의 할 수 있음.

```kotlin
var name: String = "Marcin"
var surname: String = "Moskala"
val fullName
    get() = "$name $surname"

fun main() {
    println(fullName) // Marcin Moskala
    name = "Maja"
    println(fullName) // Maja Moskala
}
```

이렇게 var 프로퍼티를 사용하는 val 프로퍼티는 var 프로퍼티가 변할 때 변할 수있음.

```kotlin
fun calculate(): Int {
    print("Calculating...")
    return 42
}

val fizz = calculate() //계산합니다...
val buzz
    get() = calculate()
fun main(){
    println(fizz)   //Calculating...42
    println(fizz)   //42
    println(buzz)   //Calculating...42
    println(buzz)   //Calculating...42
}
```

두번째 “fizz” 변수는 이미 초기화된 값이 있기 때문에 “42” 만 반환하고,

“buzz”이 경우, 호출 할 때마다 초기화를 하기 때문에 항상 “Calculating...42”이 출력 된다.

코틀린의 프로퍼티는 기본적으로 캡슐화가 되어 있고, 추가적으로 사용자 정의 접근자(getter, setter) 를 가질 수 있음. API를 변경하거나 정의할 때, 굉장히 유용.

var : setter, getter 모두 가질 수 있음.

val : 변경이 불가능하므로 getter만 가질 수 있음

그렇기 때문에 val는 var로 오버라이드 할 수 있음. (val 프로퍼티를 추가해야되기때문에 var로 오버라이딩 할수있다? 라고 이해)

```kotlin
interface Element {
    val active: Boolean
}

class ActualElement: Element {
    override val active: Boolean = false
}
```

읽기 전용 프로퍼티 val의 값은 변경될 수 있지만, 프로퍼티 레퍼런스 자체를 변경할 수는 없으므로 동기화 문제 등을 줄일 수 있음.

그래서 일반적으로 val를 많이 씀.

val은 읽기전용 프로퍼티지만, 불변(immutable)을 의미하는 것은 아님. 또한 이는 getter, delegate로 정의 할 수 있음.

getter에 의한 정의

```kotlin
class Example {
    val simpleVal: Int
        get() {
            // 사용자 정의된 getter 로직
            return 42
        }
}

fun main() {
    val example = Example()
    println(example.simpleVal) // 출력: 42
}
```

delegate에 의한 정의

```kotlin
// 델리게이트 역할을 수행할 클래스
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, delegated property name: '${property.name}'"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$thisRef, delegated property name: '${property.name}' set with value: '$value'")
    }
}

class Example {
    // "delegate" 프로퍼티에 Delegate 클래스를 델리게이트로 사용
    var delegate: String by Delegate()
}

fun main() {
    val example = Example()

    // getter 호출 (델리게이트의 getValue가 호출됨)
    println(example.delegate) // 출력: "Example@123, delegated property name: 'delegate'"

    // setter 호출 (델리게이트의 setValue가 호출됨)
    example.delegate = "New Value" // 출력: "Example@123, delegated property name: 'delegate' set with value: 'New Value'"
}
```

만약 완전히 변경할 필요가 없다면, final 프로퍼티를 이용하는것이 좋음

```kotlin
class Example2{
    private val name: String? = "maraton"
    private val surname: String = "Braun"

    val fullName: String? 
        get() = name?.let { "$it $surname" }
    
    val fullName2: String? = name?.let { "$it $surname" }
}

fun main(){

    val example2 = Example2()

    if (example2.fullName != null) {    //getter로 정의하였기때문에  스마트 캐스트 x
        println(example2.fullName.length)     //코틀린 내부에서 String? -> String 변환 x
    }

    if (example2.fullName2 != null){    //직접 변수로 정의하였기때문에  스마트 캐스트 O
        println(example2.fullName2.length)    //코틀린 내부에서 String? -> String 변환 O
    }
}
```

getter로 정의한 경우 스마트 캐스트 적용이 안됨.

직접 변수로 정의한 경우 스마트 캐스트 적용.

**가변 컬렉션과 읽기 전용 컬렉션 구분하기**

코틀린은 읽고 쓸 수 있는 컬렉션과 읽기 전용 컬렉션으로 구분.

Iterable, Collection, Set, List 인터페이스는 읽기 전용.

MutableIterable, MutableCollection, MutableSet, MutableList 는 읽고 쓸 수 있는 컬렉션.

읽기 전용 컬렉션도 대부분의 경우에는 내부의 값을 변경할 수 있지만, 읽기 전용 인터페이스가 이를 지원하지 않으므로 변경할 수 없음.

```kotlin
inline fun <T,R> Iterable<T>.map(
    transformation: (T) -> R
): List<R> {
    val list = ArrayList<R>()
    for (elem in this) {
        list.add(transformation(elem))
    }

    return list
}
```

Iterable<T>.map, Iterable<T>.filter 함수는 변경할 수 있는 리스트인 ArrayList 를 리턴함.

읽기전용 컬렉션은 Iterable<T>를 직접 변경하는건 불가능 하지만, 내부적으로는 변경 가능한 컬렉션으로 변경 후 사용하면 된다는 말이다.

이러한 컬렉션을 진짜로 불변하게 만들지 않고, 읽기 전용으로 설계하는 것은 굉장히 중요한 부분.

컬렉션을 읽 전용으로 리턴하면 이를 읽기 전용으로만 써야함.  컬렉션 다운캐스팅은 이러한 계약을 위반하고, 추상화를 무시하는 행위.

```kotlin
    val list: List<Int> = listOf(1, 2, 3)

    if(list is MutableList){
        list.add(4)
    }
    println(list)

Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
	at chapter01.Item1ExampleKt.main(Item1Example.kt:148)
	at chapter01.Item1ExampleKt.main(Item1Example.kt)
```

위와 같은 다운캐스팅은 코틀린에서 허용하지 않음. 읽기 전용 컬렉션을 mutable 컬렉션으로 다운캐시팅하면 안됨.  copy를 통해 새로운 mutable 컬렉션을 만드는 list.toMutableList를 활용해야함.

```kotlin
    val list: List<Int> = listOf(1, 2, 3)

    val mutableList = list.toMutableList()
    mutableList.add(4)
```

**데이터 클래스의 copy**

immutable 객체를 사용하면서 얻게되는 장점.

1. 한번 정의된 상태가 유지되므로, 코드를 이해하기 쉽다.
  1. 생성된 후, 그 상태가 변경되지 않으므로 코드의 흐릅을 추적하거나 디버깅에 용이.
2. immutable 객체는 공유했을 때도 충돌이 따로 이루어지지 않으므로, 병렬처리를 안전하게 할수있다.
  1. 동시에 여러 곳에서 객체를 읽어도 상태가 변하지 않기 때문에 동기화에 대한 복잡성을 줄임.
3. immutable 객체에 대한 참조는 변경되니 않으므로, 쉽게 캐시할 수 있다.
4. immutable 객체는 방어적 복사본을 만들 필요가 없다. 또한 객체를 복사할 때 깊은 복사를 따로 하지 않아도 된다.
  1. **방어적 복사본 (Defensive Copy)** 이란, 객체를 복사하여 새로운 인스턴스를 생성하여 객체의 불변성을 유지하는것
  2. **깊은 복사(Deep Copy)**는 원본 객체와 그내부의 하위 객체들을 독립적인 복사본으로 복사하는 것.
5. immutable 객체는 다른객체를 만들때 활용하기 좋음. 또한 실행을 더 쉽게 예측할 수 있음.
6. immutable 객체는 ‘set’, ‘map‘의 키로 사용 할 수 있음.

```kotlin
    val names: SortedSet<FullName> = TreeSet()
    val person = FullName("AAA","AAA")
    names.add(person)
    names.add(FullName("Jordan","Hansen"))

    println(names)
    println(person in names)  //true

    person.firstName = "ZZZ"

    println(names)
    println(person in names) //false가 나와야 되는데 실제로는 true가 나옴. 다시확인 필요

```

data 한정자는 copy라는 이름의 메서드를 만들어준다.

```kotlin
data class User(
    val name: String,
    val surname: String
)

fun main(){
    var user = User("재진", "심")
    user = user.copy(surname="박")
    print(user)   //User(name=재진, surname=박)
}
```

새로운 객체를 만들어서 일부 속성만 변경, 이것은 불변성을 지향하는 함수형 프로그래밍 스타일에 부합.

### 다른 종류의 변경 가능 지점

변경할 수 있는 리스트의 두가지 방법을 알아보자

```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()
```

두 가지 모두 변경을 할 수 있음. 방법이 다름

```kotlin
list1.add(1)
list2 = list2 + 1
```

두가지 모두 += 연산자를 이용할 수 있지만 내부 동작은 조금 다름

```kotlin
list1 += 1 //list1.plusAssign(1) 로 변경
list2 += 1 //list2 = list2.plus(1) 로 변경
```

두가지 모두 정상동작 하지만, 장단점이 있음.

첫번째 리스트

- 구체적인 구현체 내부에 변경 가능 지점이 있음. 멀티스레드 처리가 이루어질 경우, 내부적으로 적절한 동기화가 되어있는지 확실하게 알 수 없으므로 위험.

두번째 리스트

- 프로퍼티 자체가 변경 가능 지점. 따라서 멀티스레드 처리의 안정성 더 좋다고 할 수 있음.

```kotlin
//잘못 만든 경우
fun main()  {
    var list = listOf<Int>()
    for (i in 1..1000 ){
        thread{
            list = list + 1 //불변 객체에 대해 동시에 변경시도
        }
    }
    Thread.sleep(1000)
    println(list.size)  //1000 이 되지 않고, 실행할때마다 달라짐
}
//불변객체에 대해 동시에 변경시도가 있었기 때문에, 일부 요소가 손실됨.

------------------------------------------------------------------------------

//동기화 처리
fun main()  {
    var list = listOf<Int>()
    val lock = Object()
    for (i in 1..1000 ){
        thread{
            synchronized(lock) { //synchronized 블록을 이용하여 동기화 처리
                list = list + 1
            }

        }
    }
    Thread.sleep(1000)
    println(list.size)
}

//요소 손실 없음.
```

mutable 리스트 대신 mutable 프로퍼티를 사용하는 형태는 사용자 정의 setter를 활용해서 변경을 추적할 수 있음.

```kotlin
fun main() {
    var names by Delegates.observable(listOf<String>()){
        _, old, new ->
        println("Names changed from $old to $new")
    }

    names += "Fabio"
    println(names)
    names += "Bill"
    println(names)
}

--------------------------------------------

Names changed from [] to [Fabio]
[Fabio]
Names changed from [Fabio] to [Fabio, Bill]
[Fabio, Bill]
```

mutable 컬렉션을 사용하는 것이 처음에는 더 간단하게 느껴지겠지만, mutable 프로퍼티를 사용하면 객체 변경을 제어하기가 더 쉬움.

최악의 방식

```kotlin
var list3 = mutableListOf<Int>()
```

위 방법대로 프로퍼티와 컬렉션 둘다  변경 가능한 지점으로 만들게 되면, 두 지점 모두 동기화를 구현해야함.

상태를 변경하는 불필요한 방법은 만들지 말아야 함. 상태를 변경하는 모든 방법은 코드를 이해하고 유지해야 하므로 비용이 발생. 가변성을 제한하자.

### 변경 가능 지점 노출하지 말기

상태를 나타내는 mutable 객체를 외부에 노출 하는 것은 굉장히 위험.

```kotlin
data class User(
    val name: String
)

class UserRepository{
    private val storedUsers: MutableMap<Int, String> =
        mutableMapOf()

    fun loadAll(): MutableMap<Int, String> {
        return storedUsers
    }
}

fun main() {
    val userRepository = UserRepository()

    val storedUsers = userRepository.loadAll()
    storedUsers[4] = "sss"   //private은 storedUsers를 수정

    println(userRepository.loadAll()) //{4=sss}
}
```

loadAll() 를 이용해서 private 상태인 UserRepository 를 수정할 수 있음.

이를 처리하는 방법은 두 가지 있음.

**방어적 복제**

```kotlin
data class MutableUser (
    val name: String
)
class UserHolder {
    private val user =  MutableUser("sim")
    fun get(): MutableUser {
        return user.copy()    //copy를 이용해서 객체를 복제
    }

}

fun main() {
    val userHolder = UserHolder()
    println(userHolder.get())       //MutableUser(name=sim)
}
```

위와 같이 객체를 copy하여 가져오는 경우 변경 가능한 지점을 숨겨서 변경되는 것을 방지할 수 있음.

```kotlin
data class User(val name: String)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> =
        mutableMapOf()

    fun loadAll(): Map<Int,String> {  //읽기전용 슈퍼타입으로 업캐스트 하여 가변성을 제어
        return storedUsers
    }
}

fun main(){
    val storedUsers = UserRepository()
    println(storedUsers.loadAll())
}
```

### 정리

- var보다는 val를 사용하자.
- mutable 프로퍼티보다는 immutable 프로퍼티를 사용하자.
- 변경이 필요한 대상을 만들어야된다면, immutable 데이터 클래스로 만들고 copy를 활용하는 것이 좋다.
- 컬렉션의 상태를 저장해야 한다면,  mutable 컬렉션보다는 읽기 전용 컬렉션을 사용하는 편이 좋다.
- 변이 지점을 적절하게 설계하고, 불필요한 변이 지점을 만들지 않는 것이 좋다.
- mutable 객체를 외부에 노출하지 않는 것이 좋습니다.

효율성 때문에 가끔은 mutable 객체를 사용하는 것이 좋을 수 있다.

immutable 객체를 사용할 때는 언제나 멀티스레드 때에 더 많은 주의를 기울여야 한다.
