대부분의 프로그래밍 언어에는 함수 타입이라는 개념이 없어서 메소드 하나만 있는 인터페이스를 활용해서 전달한다. 이런 인터페이스를 SAM(Single-Abstract Method)라고 한다.

아래와 같이 함수는 OnClick 인터페이스를 구현한 객체를 전달받는다.

```kotlin
interface OnClick {
	fun clicked(view: View)
}

fun setOnClickListener(listener: OnClick) {~}

fun setOnClickListener(object: OnClick {
	override fun clicked(view: View) {~}
}
```

이 코드를 함수 타입을 사용하는 코드로 변경할 수 있다.

```kotlin
fun setOnClickListener(listener: (View) -> Unit) {~}
```

이를 다음과 같은 방법으로 파라미터를 전달할 수 있다.

- 람다 표현식 또는 익명함수로 전달

```kotlin
fun setOnClickListener {~} //람다 표현식

fun setOnClickListener(fun(view){~}) //익명함수
```

- 함수 레퍼런스 또는 제한된 함수 레퍼런스로 전달 (Q. 제한된 함수 레퍼런스가 뭐지? 뭐가 제한된거지?)

```kotlin
fun setOnClickListener(::println)
fun setOnClickListener(this::showUsers)
```

- 선언된 함수 타입을 구현한 객체로 전달

```kotlin
class ClickListener:(View) -> Unit {
	override fun invoke(view: View){~}
}
fun setOnClickListener(ClickListener())
```

혹자가 말하는 SAM 의 장점

1. 아규먼트에 이름을 붙일 수 있다.
    1. 반론: 타입 별칭 typealias 를 사용하면 함수 타입에 이름을 붙일 수 있다. 그러면 IDE에 hint 지원을 받을 수 있다.(Q. val를 사용해도 되는데 typealias의 장점이 뭘까?)
    
    ```kotlin
    typealias OnClick = (View) -> Unit 
    ```
    

함수 타입의 장점

1. 람다 표현식이 갖는 아규먼트 분해(destructure argument) 특징을 사용할 수 있다. SAM 은 못함 
2. 고전적인 자바 옵저버 설정 시에 발생했던 문제를 해결할 수도 있다.

```kotlin
class CalendarView{
	var listener: Listener? = null
	interface Listener {
		fun onDateClicked(date: Date)
		fun onPageChanged(date: Date)
	}
}
//->
class CalendarView{
	var onDateClicked: ((date: Date) -> Unit) ?= null
	var onPageChanged: ((date: Date) -> Unit) ?= null
}
```

onDateClicked, onPageChanged를 한꺼번에 묶지 않고 각각의 것을 독립적으로 변경할 수 있다.

( 난 이게.. SOLID 의 개방폐쇄 원칙을 지킨다고 생각함. “소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.” 고전적인 옵저버 설정은 함수 하나를 바꾼다고 한다면 다른 함수도 건드려야하는 경우가 생길 수 있으니? 인터페이스 분리 원칙과도 관련이 있어보이고!)

## 언제 SAM을 사용할까?

코틀린이 아닌 다른 언어에서 사용할 클래스를 설계할 때 SAM 이 유용하다. 함수 타입으로 만들어진 클래스는 자바에서 타입 별칭과 IDE의 지원을 제대로 받지 못하기 때문이다. 그리고 Unit을 명시적으로 리턴하는 함수가 필요하다.

```java
new CalendarView().setOnDateClicked(date -> Unit.INSTANCE);
```
