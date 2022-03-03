지켜야 하는 이유

1. 쉽게 이해할 수 있다
2. 코드를 병합하고, 한 프로젝트의 코드 일부를 다른 코드로 이동하는 것이 쉽다.

도구

1. IntelliJ formmater (Settings → Editor → Code Style → Kotlin  → Set from.. → Predefined style/Kotlin style guild
2. ktlinkn; 린터 
3. 코틀린 문서 Coding Convention

```kotlin
//비추
class Person(val id: Int=0,
val name:String = "",
val surname: String = "") : Human(id, name){~}

//추
class Person(
val id: Int=0,
val name:String = "",
val surname: String = ""
) : Human(id, name){~}
```

문제의 소지

1. 모든 클래스의 아규먼트가 클래스 이름에 따라서 다른 크기의 들여쓰기를 갖는다. 클래스 이름을 변경하면 모든 생성자의 들여쓰기를 조정해야한다.
2. 클래스가 차지하는 공간의 너비와 이름이 가장 긴 파라미터와 슈퍼클래스 지정이 있는 줄도 너무 크다.

프로젝트의 코드는 마치 한 사람이 작성한 것처럼 작성되어야 한다.
