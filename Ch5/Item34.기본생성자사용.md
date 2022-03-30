객체를 정의하고 생성하는 가장 기본적인 방법은 기본 생성자를 사용하는 것이다. 초기 상태를 나타내는 아규먼트를 전달해서 초기화한 후 그 프로퍼티를 유지한다. 

다음 예제는 MVP(Model-View-Presenter) 아키텍처에서 사용되는 프레젠터 객체에서 기본 생성자를 사용해 종속성을 주입하는 예제이다.

```kotlin
class QuotationPresenter(
	private val view: QuotationView,
	private val repo: QuotationRepository
) {
	private var nextQuoteId = -1
	fun onStart() { onNext() }
	fun onNext() {
		nextQuoteId = (nextQuoteId + 1) % repo.quotesNumber
		val quote = repo.getQuote(nextQuoteId)
		view.showQuote(quote)
	}
}
```

QuotationPresenter는 기본 생성자에 선언되어 있는 프로퍼티보다 더 많은 프로퍼티를 갖고 있고 초기화된다. 기본 생성자가 좋은 방식을 이해하기 위해 먼저 다음 두 가지 자바 패턴들을 이해하는 것이 좋다.

1. 점층적 생성자 패턴
2. 빌더 패턴

## 34.1. 점층적 생성자 패턴

여러 가지 종류의 생성자를 사용하는 패턴이다.

```kotlin
class Pizza {
	val size: String
	val cheese: Int
	val olives: Int
	val bacon: Int

	constructor(size: String, cheese: Int, olives:Int, bacon: Int) {~}
	constructor(size: String, cheese: Int, olives: Int):this(size, cheese, olives, 0)
..
}
```

이 코드는 그리 좋은 코드가 아니고 디폴트 아규먼트를 사용하면 더 깔끔하게 같은 기능을 제공할 수 있다.

```kotlin
class Pizza (
	val size: String
	val cheese: Int = 0,
	val olives: Int = 0,
	val bacon: Int = 0

)

val myFavorite = Pizza("L", olives = 3)
val myFavorite2 = Pizza("L", olives 3, cheese = 1)
```

이렇게 하면 장점이 

1. 파라미터들의 값을 원하는 대로 지정할 수 있다.
2. 아규먼트를 원하는 순서로 지정할 수 있다.
3. 명시적으로 이름을 붙여서 아규먼트를 지정하기 때문에 의미가 훨씬 명확하다. 이 항목이 가장 중요하다.

## 34.2. 빌더 패턴

**장점**

1. 파라미터에 이름을 붙일 수 있다.
2. 파라미터를 원하는 순서로 지정할 수 있다.
3. 디폴트 값을 지정할 수 있다.

```kotlin
class Pizza private constructor(
	val size: String,
	val cheese: Int,
	val olives: Int,
	val bacon: Int
) {
	class Builder(private val size: String) {
		private var cheese: Int = 0
		private var olives: Int = 0
		private var bacon: Int = 0
	
		fun setCheese(value: Int): Builder = apply {
			cheese = value
		}

		fun setOlives(value: Int): Builder = apply {
			olives = value
		}

		fun setBacon(value: Int): Builder = apply {
			bacon = value
		}

		fun build() = Pizza(size, cheese, olives, bacon)
	}

}

val myFavorite = Pizza.Builder("L").setOlives(3).build()
```

장점이 기본 생성자의 이름 있는 파라미터를 사용하는 것과 비슷하다. 

빌더 패턴을 사용하는 것 보다 이름 있는 파라미터를 사용하는 것이 좋은 이유는 4가지가 있다.

1. 더 짧다: 디폴트 아규먼트가 있는 생성자, 팩토리 메소드가 빌드 패턴보다 구현과 이해가 쉽다. 또 수정을 할 때에도 생성자 파라미터, 함수의 이름, 파라미터 이름, 본문, 내부 필드등을 모두 변경해야 한다.
2. 더 명확하다: 객체가 어떻게 생성되는지 보고 싶을 때 빌더 패턴은 여러 메소드를 확인해야 하지만 디폴트 아규먼트가 있는 코드는 생성자 부분만 보면 된다.
3. 더 사용하기 쉽다: 기본 생성자는 언어에 내장된 개념이지만 빌더 패턴은 언어 위에 추가로 구현한 개념이기 때문에 knowledge가 필요하다.
4. 동시성과 관련된 문제가 없다: 코틀린의 함수 파라미터는 immutable이디만 빌더 패턴의 프로퍼티는 mutable이기 떄문에 thread-safe하게 구현하기 어렵다.

하지만 기본 생성자 대신에 빌더 패턴이 유용할 때도 있다. 빌더 패턴은 값의 의미를 묶어서 지정할 수 있다.

```kotlin
val dialog = AlertDialog.Builder(context)
						.setMessage(R.String.file_missiles)
						.setPositiveButton(R.string.file, {d, id ->~})
						.create()

// 빌더 패턴을 쓰지 않으면 
val dialog = AlertDialog(context, message = R.string.file_misiles, 
												positiveButtonDescription = ButtonDescription(R.string.fire, {d, id-> ~ })
```

하지만 이것도 DSL(DOmain Specific Language) 빌더를 사용한다.

```kotlin
val dialog = context.alert(R.string.file_missiles) { 
	positiveButton(R.string.file) {~}
}
```

DSL 빌더 패턴은 전통적인 빌더 패턴보다 훨씬 유연하고 명확해서 많이 사용하지만 만드는 것이 조금 어렵지만 보다 유연하고 가독성이 좋다(왜 더 유연할까(?). 이는 다음 장에서 더 알아보자. 

전통적인 빌더 패턴의 또 다른 장점은 팩토리로 사용할 수 있다는 점이다. 팩토리 메소드를 기본 생성자처럼 사용하려면 커링을 활용해야 하지만 코틀린은 커링을 지원하지 않는다. 대신에 데이터 클래스를 만들고 copy로 객체를 복제한 뒤, 필요한 설정을 수정하는 형태로 만든다.

(코틀린..커링 지원하는데..[https://zerogdev.blogspot.com/2020/02/kotlinhighfunc.html](https://zerogdev.blogspot.com/2020/02/kotlinhighfunc.html))

```kotlin
data class DialogConfig(val icon: Int = -1,
val title: Int = -1,
val onCancelListener: (() -> Unit)? = null)

fun makeDefaultDialogConfig() = DialogConfig(
icon = R.drawable.ic_dialog,
title = R.string.dialog_title,
onCancelListener = { it.cancel() }
)
```

하지만 코틀린에서는 빌더 패턴을 거의 사용하지 않고, 다음과 같은 경우에만 사용한다.

1. 빌더 패턴을 사용하는 다른 언어로 작성된 라이브러리를 그대로 옮길 때 
2. 디폴트 아규먼트와 DSL을 지원하지 않는 다른 언어에서 쉽게 사용할 수 있게 API를 설계할 때 

이 때를 제외하면 빌더 패턴 대신에 디폴트 아규먼트를 갖는 기본 생성자나 DSL을 사용한다.

## 34.3. 정리

일반적인 프로젝트에서는 기본 생성자를 사용해 객체를 만든다. 점층적 생성자 패턴을 사용하지 않고 디폴트 아규먼트를 활용하는 것이 더 좋다.
