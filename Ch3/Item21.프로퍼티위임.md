# 아이템21. 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

**프로퍼티 위임(property delegate** 는 일반적인 프로퍼티의 행위를 추출해서 재사용할 수 있다. 예를 들어 지연 프로퍼티(lazy property)는 굉장히 많이 사용되는데 필요할 때마다 구현할 필요 없이 프로퍼티 위임을 쉽게 구현하게 하는 lazy 함수를 사용해 쉽게 구현할 수 있다.

```kotlin
val value by lazy { createValue() }
```

변화를 감지하는 observable 패턴도 쉽게 만들 수 있다.

```kotlin
var items: List<Item> by Delegates.observable(listOf()) {_,_,_ -> notifyDataSetChanged() }
```

뷰, 리소스 바인딩, 의존성 주입, 데이터 바인딩등의 다양한 패턴을 어노테이션 없이 프로퍼티 위임을 통해 만들 수 있다.

```kotlin
//안드로이드 뷰, 리소스 바인딩
private val button: Button by bindBiew(R.id.button)
private val textSize by bindDimension(R.dimen.font_size)
private val doctor: Docter by argExtra(DOCTOR_ARG)

//kotlin 의존성 주입
private val presenter: MainPresenter by inject()
private val repository: NetworkRepository by inject()
private val vm: MainViewModel by viewModel()

//데이터 바인딩
private val port by bindConfiguration("port")
private val token:String by preferences.bind(TOKEN_KEY)
```

일부 프로퍼티가 사용될 때 간단한 로그를 출력한다면?

```kotlin
//1. 게터 세터
var token: String? = null
	get() {
		print("token returned value $field")
		return field
	}
	set(value) {
		print("token changed from $field to $value")
		field = value
	}

//2. 프로퍼티 위임 
var token: String? by LoggingProperty(null)

private class LoggingProperty<T>(var value: T) {
	operator fun getValue(
		thisRef: Any?,
		prop: KProperty<*>
	): T {
		print("${prop.name} returned value $value")
		return value
	}

	operator fun setValue(
		thisRef:Any?,
		prop: KProperty<*>,
		newValue: T
	) {
		val name = prop.name
		print("$name changed from $value to $newValue")
		value = newValue
	}
}

//2.1 by 컴파일
@JvmField
private val `token$delegate` = LoggingProperty<String?>(null)
val token: String? 
	get() = `token$delegate`.getValue(this, ::token)
	set(value) {
		`token$delegate`.setValue(this, ::token, value)
	}
```

getValue, setValue는 단순 값 처리만 하는게 아니라 컨텍스트(this)와 프로퍼티 레퍼런스의 경계도 함께 사용하는 형태로 바뀐다.(?) getValue, setValue가 여러 개 있어도 컨텍스트를 활용하기 때문에 적절한 메소드가 선택된다.

객체를 프로퍼티 위임하려면 val 의 경우 getValue, var의 경우 getValue, setValue 연산이 필요하고, 이는 확장함수로도 만들 수 있다.

```kotlin
inline operator fun <V, V1:V> Map<in spring, V>.getValue(thisRef: Any?, property: KProperty<*>): V1
= getOrImplicitDefault(property.name) as V1
```

stdlib 에서 다음과 같은 프로퍼티 델리게이터를 알아 두면 좋다.

- lazy
- Delegates.observerble
- Delegates.vetoable
- Delegates.notNull
