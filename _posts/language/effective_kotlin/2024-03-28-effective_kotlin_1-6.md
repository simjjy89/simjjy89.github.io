---
title:  "1장 안전성 item 6"
excerpt: " 사용자 정의 오류보다는 표준 오류를 사용하라."

categories:
  - effective_kotlin
tags:
  - [kotlin]

toc: true
toc_sticky: true
 
date: 2024-03-28
last_modified_at: 2024-03-28
---

### 1장 안정성_ 사용자 정의 오류보다는 표준 오류를 사용하라

가능하다면, 사용자 정의 오류보다는 최대한 표준 라이브러리의 오류를 사용하는 것이 좋다.

많은 개발자들이 알고 있으므로, 이를 재사용 하는 것이 좋다.

```kotlin
inline fun <reified T> String.readObject(): T {
    //...
    if (incorrectSign) {
        throw JsonParsingException()
    }
    //...
    return result
}
```

표준 라이브러리에 적절한 오류가 없어서 사용자 정의 오류를 사용했지만, 가능하다면 표준 라이브러리의 오류를 사용하는 것이 좋다.

- **IllegalArgumentException**과 **IlleagalStateException**
- **IndexOutOfBoundsException**
- **ConcurrentModificationException**
- **UnsupportedOperationException**
- **NoSuchElementExceptoin**