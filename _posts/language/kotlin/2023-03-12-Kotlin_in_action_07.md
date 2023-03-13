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

| 식     | 함수 이름           |
| ----- | --------------- |
| a * b | times           |
| a / b | div             |
| a % b | mod(1.1 부터 rem) |
| a + b | plus            |
| a - b | minus           |

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

### 복합 대입 연산자 오버로딩

plus와 같은 연산자를 오버로딩하면 코틀린은 그와 관련있는 복합대입 연산자도 함께 지원함

```kotlin
var point = Point(1,2)
point += Point(3,4)

println(point)
---------------------------------------------
Point(x=4, y=6)
```

반환타입이 Unit인 plussAssign 함수를 정의하면 코틀린은 += 연산자에 그 함수를 사용한다. 

코틀린 표준 라이브러리는 변경 가능한 컬렉션에 대해 plusAssign을 정의함.

```kotlin
operator  fun <T> MutableCollection<T>.plusAssign(element: T) {
    this.add(element)
}
```

+와 -는 항상 새로운 컬렉션을 반환하며, +=와 -= 연산자는 항상 변경 가능한 컬렉션에 작용해 메모리에 있는 객체 상태를 변화시킴. 

또한, 읽기전용 컬렉션에서 +=, -=는 변경을 적용한 복사본을 반환.

```kotlin
    val list = arrayListOf(1,2) //가변 리스트
    println(System.identityHashCode(list))
    list+=3
    println(System.identityHashCode(list))
    val newList = list+ listOf(4,5) //읽기전용 리스트
    println(System.identityHashCode(newList))
    println(newList)  
    
-----------------------------------------------------------
    
1845066581
1845066581
2136344592
[1, 2, 3, 4, 5]
```



### 단항 연산자 오버로딩

```kotlin
operator fun Point.unaryMinus() : Point{ //단항 함수는 파라미터가 없다
    return Point(-x,-y) //좌표에서 각 성분의 음수를 취한 새 점을 반환
}

val p1 = Point(10, 20)
println(-p1)
--------------------------------------------------------
Point(x=-10, y=-20)
```



### 비교 연산자 오버로딩

코틀린에서는 원시타입뿐만 아니라 모든 객체에 대해 비교 연산을 수행 할 수 있음.



##### 동등 연산자 : equals

- == 연산자 호출은 equals 메소드 호출러 컴파일됨

- != 연산자를 사용하는 식도 equals 호출로 컴파일.



##### 순서 연산자 : compareTo

 