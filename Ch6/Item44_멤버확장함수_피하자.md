확장 함수를 클래스의 멤버로 정의하는 것은 좋지 못하다.

1. 가시성을 제한하지 못한다.
2. 레퍼런스를 지원하지 않는다.

```kotlin
class {
	fun String.isPhoneNumber() = ~
}

val str = "1234"
val boundedRef = str::isPhoneNumber //오류
```

1. 암묵적 접근을 할 떄, 두 리시버 중 어떤 리시버가 선택될지 혼동된다.

```kotlin
class A { 
	val a = 10
}

class B {
	val a = 20
	val b = 30

	fun A.test() = a+b //일 때 a는 10,20중에 혼동이 된다.
}
```

1. 가독성이 좋지 않다.
