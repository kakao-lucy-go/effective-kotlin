상수(constant)모드를 가진 클래스가 있다. 상수 모드를 모두 태그(tag)라고 하고 태그를 포함한 클래스를 태그 클래스(tagged class)라고 한다. 

태그 클래스는 서로 다른 책임을 한 클래스에 태그로 구분해서 넣는다는 것에서 문제가 시작한다.

```kotlin
class ValueMatcher<T> private constructor(
	private val value: T? =null,
	private val matcher: Matcher
){
	fun match(value: T?) = when(matcher) {
		Matcher.EQUAL -> value == this.value
		Mathcer.NOT_EQUAL -> value != this.value
	}

	enum class Matcher {
		EQUAL, NOT_EQUAL
	}
}
...
```

이 방식의 단점

1. 한 클래스에 여러 모드를 처리하기 위한 사용구가 추가된다.(match 함수)
2. 여러 목적에서 사용하기 때문에 일관적이지 않게 사용할 수 있다. 예를 들어 value는 어떤 모드에서는 아예 사용되지 않을 수 있다.
3. 요소가 여러 목적을 가지고 여러 방법으로 설정할 수 있을 때 일관성과 정확성을 지키기 어렵다.
4. 팩토리 메소드를 사용해야 하는 경우가 많다. 그렇지 않으면 객체가 제대로 생성됐는지 확인하는 것이 어렵다.

따라서 보통 태그 클래스보다 sealed 클래스를 더 많이 사용한다.

한 클래스에서 여러 모드를 만드는 것이 아니라, 각각의 모드를 여러 클래스로 만들어서 타입 시스템과 다형성을 활용하는 것이다.

```kotlin
sealed class ValueMatcher<T> {
	abstract fun match(value: T): Boolean

	class Equal<T>(val value:T): ValueMatcher<T>() {
		override fun match(value: T): Boolean = value == this.value
	}
}
```

이러면 책임이 분산되고, 각각의 객체는 자신에게 필요한 데이터만 있으며, 적절한 파라미터만 갖는다.

## sealed 한정자

abstract 한정자도 있지만 sealed 클래스는 외부에서 서브 클래스를 만드는 행위를 제한하기 때문에 타입이 추가되지 않는게 보장이 된다. 따라서 when 을 사용할 때에도 else를 사용하지 않아도 된다.

abstract를 사용하면 when 을 사용할 때 외부에서 새로운 클래스가 추가되면 함수가 제대로 동작하지 않을 수 있다.

## 태그 클래스와 상태 패턴의 차이

상태 패턴은 객체의 내부 상태가 변화할 때 객체의 동작이 변하는 디자인 패턴이다. controller, presenter, view (MVC, MVP, MVVM 아키텍처)를 설계할 때 많이 사용한다.

```kotlin
sealed class WorkoutState

class PrepareState(val exercise: Exercise): WorkoutState()

class ExerciseState(val exercise: Exercise): WorkoutState()

object DoneState: WorkoutState()

fun List<Exercise>.toStates(): List<WorkoutState> = ~

class WorkoutPresenter(~) {
	private var state: WorkoutState = states.first()
}
```

1. 상태는 태그 클래스보다 더 많은 책임을 가진 큰 클래스다.
2. 상태는 변경할 수 있다.

concreate state는 객체를 활용해서 표현하는 것이 일반적이고 태그 클래스보다 sealed 클래스로 계층 구조를 만든다. 이를 immutable 객체로 만들어서 state 프로퍼티를 변경하고 뷰에서 이를 관찰한다.

## 정리

코틀린은 태그 클래스보다 sealed를 활용한 타입 계층을 사용하는 것이 좋다.
