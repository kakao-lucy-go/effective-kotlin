# 아이템 47. 인라인 클래스의 사용을 고려하라

인라인은 함수 뿐 아니라 하나의 값을 보유하는 객체도 inline으로 만들 수 있다. 기본 생성자 프로퍼티가 하나인 클래스 앞에 inline을 붙이면 해당 객체를 사용하는 위치가 모두 해당 프로퍼티로 교체된다.

```kotlin
inline class Name(private val value: String) {
	fun greet() {println("Hello $value")}
}

//compile
val name:String = "Marcin"
Name.'greet-impl'(name)
```

inline 클래스의 메소드는 전부 정적 메소드로 만들어진다.

다른 자료형을 래핑해서 새로운 자료형을 만들 때 많이 사용한다. (String을 Name으로 래핑) 이 때, 어떠한 오버헤드가 발생하지 않는다.

주로,

- 측정 단위를 표현할 때
- 타입 오용으로 발생하는 문제를 막을 때

사용한다.

## 측정 단위를 표현할 때

```kotlin
interface Timer{
	fun callAfter(time: Int, callback: ()->Unit)
}
```

이 때, time의 단위를 알기 어려우므로 다음과 같이 할 수 있다.

```kotlin
interface Timer{
	fun callAfter(timeMillis: Int, callback: ()->Unit)
}
```

하지만 리턴할 경우 단위를 알기 어려운 단점이 있다. 함수에 이름을 붙일 수 도 있지만 더 좋은 것은 타입에 제한을 거는 것이다.

```kotlin
inline class Millis(val milliseconds: Int) {
	
}

interface Timer {
	fun callAfter(timeMillis: Millis, callback: ()->Unit)
}

```

이러면 타입이 강제된다. 프론트에서는 px, mm, dp 등의 다양한 단위를 사용하는데 이를 제한할 때 사용하면 좋다.

## 타입 오용으로 발생하는 문제를 막을 때

SQL DB에서는 보통 ID로 요소를 식별하고, Int를 자료형으로 쓰는 경우 잘못된 값을 넣을 수도 있다. 이때 Int 를 inline 클래스를 활용하면 좋다.

```kotlin
inline class StudentId(val studentId: Int)

class Grades(
	@ColumnInfo(names = "studentId")
	val studentId: StudentId
)
```

이러면 컴파일할 때 타입이 Int로 대체되기 때문에 안전하면서 문제의 소지를 줄여줄 수 있다.

## 인라인 클래스와 인터페이스

인라인 클래스도 인터페이스를 구현할 수 있다.

```kotlin
interface TimeUnit {
	val millis: Long
}

inline class Minutes(val minutes: Long): TimeUnit {
	override val millis: Long get() = minutes * 60 * 1000
}

```

하지만 이 Minutes을 사용하더라도 inline으로 동작하진 않는다. 인터페이스를 통해서 타입을 나타내려면 객체를 래핑해서 사용해야 하기 때문이다.

## typealias

타입에 이름을 붙여줄 수 있는 typealias로 단위를 표현하려고 하면 이는 안전하지 않는다. 

```kotlin
typealias Seconds = Int 
typealias Millis = Int
fun getTIme(): Millis = 10
fun setUpTimer(time: Seconds){}

setUpTimer(getTime())
```

따라서 단위를 나타내려면 파라미터 이름 혹은 클래스를 사용한다.

파라미터 이름은 비용이 적게들고, 클래스는 안전해진다. 인라인 클래스를 사용하면 비용도 적게 들고 클래스도 안전해진다.

## 정리

인라인 클래스는 성능 오버헤드 없이 타입을 래핑할 수 있다.
