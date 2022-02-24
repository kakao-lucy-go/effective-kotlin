# 아이템8. 적절하게 null을 처리하라

함수가 null 을 리턴한다는 것은 여러 의미를 가진다. 예를 들어, 아래 2가지가 있다.

1. String.toIntOrNull()은 String을 Int로 적절하게 변환할 수 없는 경우 반환
2. Iterable<T>.firstOrNull() 주어진 조건에 맞는 요소가 없는 경우 반환 

nullable 타입을 처리하는 방법은 보통 아래 3가지가 있다.

1. ?., 스마트캐스팅, Elvis 연산자등을 활용해서 안전하게 처리
2. 오류를 throw
3. 리팩토링해서 nullable 타입이 나지 않게 처리한다.

### 1.8.1 null을 안전하게 처리하기

```kotlin
printer?.println() //안전 호출
if(printer != null) printer.print() //스마트 캐스팅
val printer1 = printer?.name ?: "Unnamed" //Elvis 연산자 1
val printer2 = printer?.name ?: return //Elvis 연산자 2
val printer3 = printer?.name ?: throw Error("Printer must be named") //Elvis 연산자 3
```

- 방어적 프로그래밍 : 모든 가능성을 올바른 방식으로 처리하는 것. 프로덕션 환경으로 들어갔을 때 발생하는 수많은 것들로부터 프로그램을 방어해서 안정성을 높인다.
- 공격적 프로그래밍 : 하지만 모든 상황을 안전하게 처리하는 것은 불가능하고, 개발자에게 알려서 수정하게 만드는 것으로, 아이템 5에서 본 것 처럼 require, check, assert를 사용해서 예외 코드에 제한을 건다.
    
    

### 1.8.2 오류 throw

당연히 그럴것이라고 생각한 부분에서 문제가 발생하는 경우에 throw, !!, requireNotNull, checkNotNull 등을 활용해서 개발자에게 오류를 강제로 발생시킨다.

```kotlin
fun process(user: User) {
	requireNotNull(user.name)
	val context = checkNotNull(context)
	val networkService = getNetworkService(context) ?: throw NoInternetConnection()
	networkService.getData { data, userData -> show(data!!, userData!!) }
}
```

### 1.8.3 not-null assertion(!!) 과 관련된 문제

어떤 대상이 null 이 아니라고 생각하고 다루면 NPE 가 발생하기 떄문에 좋은 해결방법은 아니다. nullable이지만 null 이 나오지 않는 다는 것이 확실한 상황에서 많이 사용되는데 좋지 않다.

이렇게 작성하면 프로퍼티를 언팩(unpack)해야 하므로 사용하기 귀찮고 추후에 의미있는 null 값을 가질 가능성을 차단하기 때문에 지양해야 한다.

이럴 땐 나중에 살펴볼 lateinit, Delegates.notNull 을 사용한다.

명시적으로 예외를 발생하도록 설계하려면 아이템 7. null 과 Failure를 참조한다. NPE보다 많은 정보를 줄 수 있기때문에 좋다.

!!는 일반적으로 nullableility가 제대로 표현되지 않는 라이브러리에 사용하고, 그렇지 않다면 피해야 한다.

### 1.8.4 의미없는 nullability 피하기

꼭 필요한 경우가 아니라면 nullability를 피한다. null 이 중요한 메시지로 사용될 수 있기 때문이다.

- 아이템 7. null 과 Failure를 참조해서 getOrNull 과 같은 함수를 사용한다.
- 클래스 생성 이후에 확실하게 설정된다는 보장이 된다면 lateinit 이나 notNull 델리게이트를 사용한다.

```kotlin
var max: Int by Delegates.notNull()

// println(max) // will fail with IllegalStateException

max = 10
println(max) // 10
//참조 https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/not-null.html 
```

- 요소가 부족하다면 빈 컬렉션을 사용한다.
- nullable enum 과 None enum은 다르다. null enum 은 별도로 처리해야하지만 none enum 은 정의에 없어서 필요한 경우에 사용하는 쪽에서 추가해서 활용할 수 있다는 의미이다.

### 1.8.5 lateinit 프로퍼티와 notNull 델리게이트

클래스가 생성 중에 초기화할 수 없는 프로퍼티를 가지는 경우(Junit의 @BeforeEach 같이) nullable로 선지정하고 null 이 아닌 것으로 타입 변환하는 것은 좋지 않다. 이럴 때 lateinit 한정자를 사용해서 프로퍼티가 이후에 설정된다는 것을 명시한다.

```kotlin
class UserControllerTest {
	private lateinit var dao: UserDao
	private lateinit var controller: UserController

	@BeforeEach
	fun init() {
		dao = mockk()
	controller = UserController(dao)
	}
}
```

단, 초기화 전에 값을 사용하려고 하면 예외가 발생한다.

**lateinit 과 nullable의 차이**

- lateinit은 !! 연산자로 언팩하지 않아도 사용할 수 있다.
- 추후에 null을 사용하고 싶을 때 nullable로 만들 수 있다.
- 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로 돌아갈 수 없다.

lateinit은 반드시 초기화될 거라고 예상되는 상황에 활용한다. 반대로 사용하지 못하는 경우에는 JVM에서 Int, Long, Double, Boolean과 같은 기본 타입과 연결된 타입으로 프로퍼티를 초기화해야하는 경우에 사용한다. 이런 경우에는 Delegates.notNull을 사용한다.

```kotlin
private var docterId: Int by Delegates.notNull()

private var docterId; Int by arg(DOCTOR_ID_ARG)
```

자세한 건 ‘아이템 21: 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라’ 참조
