모듈의 다양한 구성 요소인 클래스, 객체, 함수, 타입 별칭, 톱 레벨 프로퍼티 등의 일부는 var나 mutable을 사용하면 상태(state)를 가질 수 있다. 

```kotlin
var a = 10
var list: MutableList<Int> = mutableListOf()
```

유용하긴 하지만 상태를 관리하는 것은 어려운 일이다. 상태를 갖게 되면,

1. 프로그램을 이해하고 디버그하기 힘들어진다. 상태를 갖는 부분들의 관계를 이해해야 하고, 변경을 추적해야 하기 때문에 이후에 코드를 수정하기 어렵고 예기치 못한 오류를 발생시키기도 한다.
2. 가변성(mutability)이 있으면 코드의 실행을 추론하기 어려워진다. 시점에 따라서 값이 달라질 수도 있기 떄문에 어느 시점에 확인한 값이 계속 동일하게 유지된다고 확신할 수 없다.
3. 멀티쓰레드 프로그램일 때에는 적절한 동기화가 필요하다. 변경이 일어나는 모든 부분에 충돌이 일어날 수 있다.
4. 테스트가 어렵다. 발생할 수 있는 모든 상태에 대해 테스트해야하기 때문에 테스트 조합 경우의 수가 많아진다.
5. 상태 변경이 일어날 때 다른 부분에 알려야 하는 경우도 존재한다. 

예를 들어서, 멀티쓰레드로 프로퍼티를 수정하는 경우 충돌에 의해 일부 연산이 이루어지지 않는다. 아래 예제는 실행할 때마다 다른 값이 나온다.

```kotlin
var num = 0
for(i in 1..1000) {
    thread {
				println(i)
        Thread.sleep(10)
        num += 1
    }
}

Thread.sleep(5000)
println(num)

//num 결과값이 다르게 나온다.
//978
//985

//thread 안의 println
16
4
11
24
5
22
15
8
...
```

코루틴을 사용하면 더 적은 스레드가 관여하기 때문에 충돌 문제가 줄어들지만 문제가 사라지는 것은 아니다. 동기화를 사용해서 문제를 해결할 수도 있지만 변경 가능 지점이 많아져서 동기화할 곳이 많아지면 그만큼 복잡해진다.

‘하스켈'이라는 순수 함수형 언어처럼 가변성에 제한을 두는 언어도 있기는 하지만 주류로 사용되지는 않는다.

## 1.1 코틀린에서 가변성 제한하기

### 1.1.1 읽기 전용 프로퍼티 (val)

일반적인 방법으로는 값이 변하지 않는다. 하지만 완전히 변경 불가능한 것은 아니고 mutable 객체를 담고 있다면 내부적으로 변할 수 있다.

```kotlin
val list = mutableListOf(1,2,3)
        list.add(4)
        println(list)

//[1, 2, 3, 4]
```

읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의해서 다른 프로퍼티가 변할 때 같이 변할 수 있다.

```kotlin
var name = "Marcin"
    var surname = "Moskala"
    val fullName
        get() = "$name $surname"

    @Test
    fun mutable() {
        println(fullName)
        name = "Maja"
        println(fullName)
    }

//Marcin Moskala
//Maja Moskala
```

코틀린의 프로퍼티는 기본적으로 캡슐화되어 있고, 추가적인 사용자 정의 접근자(게터, 세터)를 가질 수 있는데 이는 API를 변경하거나 정의할 때 유연하다. 이는 아이템 16에서 구체적으로 설명하겠다.

var는 게터와 세터 모두 제공하지만 val은 변경 불가능하기 때문에 게터만 제공한다. 그리고 val을 var로 오버라이드할 수 있다.

```kotlin
interface Element {
    val active: Boolean
}

class ActualElement: Element {
    override var active: Boolean = false
}
// 책의 예제는 interface 의 active가 var 인데 오타인듯. val!
```

val의 값은 변경될 수 있긴 하지만 프로퍼티 레퍼런스 자체를 변경할 수는 없어서 동기화 문제를 줄일수 있기 때문에 일반적으로 var 보다 val 을 주로 사용한다.

val 은 읽기 전용 프로퍼티지만 변경할 수 없는 것(immutable)을 의미하는 것은 아니다. 이는 게터 또는 델리게이트로 정의할 수 있다. 만약 완전히 변경할 필요가 없다면 final 프로퍼티를 사용하는 것이 좋다. 이때 스마트 캐스트 등의 추가적인 기능을 활용할 수 있다.

```kotlin
var name : String? = "Marcin"
var surname : String = "Moskala"
val fullName1: String?
    get() = name?.let { "$it $surname" }

val fullName2: String? = name?.let { "$it $surname" } //final 이고 사용자 정의 게터를 갖지 않는다.

@Test
fun smartCast() {
    if(fullName1 != null) {
        println(fullName1.length) //오류. 게터로 정의한 값은 데이터가 변경될 수 있기 때문에 스마트캐스트 불가
    }
    
    if(fullName2 != null) {
        println(fullName2.length) //String? 가 String 으로 스마트캐스트
    }
}
```

### 1.1.2 가변 컬렉션과 읽기 전용 컬렉션 구분하기

코틀린은 읽고 쓸 수 있는 프로퍼티와 읽기 전용 프로퍼티로 구분된다. (참조 [https://github.com/goyanglee/kotlin_in_action/blob/main/2장. 코틀린 기초/2장 코틀린 기초.md](https://github.com/goyanglee/kotlin_in_action/blob/main/2%EC%9E%A5.%20%EC%BD%94%ED%8B%80%EB%A6%B0%20%EA%B8%B0%EC%B4%88/2%EC%9E%A5%20%EC%BD%94%ED%8B%80%EB%A6%B0%20%EA%B8%B0%EC%B4%88.md))

그림 참조: [https://0391kjy.tistory.com/58](https://0391kjy.tistory.com/58) 

mutable 이 붙은 인터페이스는 대응되는 읽기 전용 인터페이스를 상속 받아서 변경을 위한 메소드를 추가한 방식이다. 읽기 전용 컬렉션이 내부 값을 변경할 수 없다는 의미는 아니지만 인터페이스에서 지원하지 않기 때문에 변경할 수 없다. 아래는 map의 구현을 간략하게 표현한 코드다.

```kotlin
inline fun <T, R> Iterable<T>.map (
	transformation: (T) -> R
): List<R> {
	val list = ArrayList<R>()
	for(elem in this) {
		list.add(transformation(elem))
	}
}
```

진짜 불변으로 만드는게 아니라 읽기 전용으로 설계한 중요한 부분이다. immutable하지 않은 컬렉션을 외부에서 보기에 immutable하게 함으로서 얻은 안정성이지만 다운캐스팅을 하는 경우 계약(2부 참고) 위반과 추상화를 위반하는 문제를 발생시켜 안전하지 않게 된다.

아래는 JVM의 경우에는 Array.ArrayList가 반환되고 코틀린의 MutableList로 변경은 할 수 있지만 Arrays.ArrayList는 add, set을 구현하고 있지 않아 예외가 발생한다.

```kotlin
val list = listOf(1,2,3)

if(list is MutableList) {
	list.add(4) 
  //list 에 형광펜 칠해지면서 Smart cast to kotlin.collections.MutableList<kotlin.Int> 표기
  // java.lang.UnsupportedOperationException 예외 발생
}

```

따라서 읽기 전용에서 mutable로 변경해야한다면 copy 를 사용해서 새로운 mutable을 만드는 list.toMutableList를 활용해야 한다.

```kotlin
val list = listOf(1,2,3)
val mutableList = list.toMutableList()
mutableList.add(4)
println(mutableList)
//[1, 2, 3, 4]
```

### 1.1.3 데이터 클래스의 copy 사용

String이나 Int 처럼 내부 상태를 변경하지 않는 immutable 객체를 많이 사용하는 데에는 이유가 있다.

1. 한번 정의된 상태가 유지되므로 코드를 이해하기 쉽다.
2. immutable 객체는 공유했을 때 충돌이 따로 이루어지지 않으므로 병렬 처리를 안전하게 할 수 있다.
3. immutable 객체에 대한 참조는 변경되지 않아서 쉽게 캐시할 수 있다.
4. immutable 객체는 방어적 복사본을 만들 필요가 없다.
5. immutable 객체는 다른 객체(Mutable, Immutable)를 만들 때 활용하기 좋고 immutable은 실행을 더 쉽게 예측할 수 있다.
6. immutable 객체는 Set, Map의 key 로 사용될 수 있다.

Immutable인 Int도 plus, minus 메소드로 새로운 Int를 리턴할 수 있고 Iterable도 map이나 filter로 새로운 Iterable을 만들어서 리턴한다. 예를 들어 우리가 User 라는 immutable 객체를 만들어도 새로운 메소드를 제공해서 자신을 수정한 새로운 객체를 만들어낼 수 있게 해야한다.

모든 프로퍼티에 대해 하나하나 만드는 것도 귀찮은 일이니까 data 클래스의 copy 라는 메소드를 사용하면 모든 기본 생성자 프로퍼티가 같은 새로운 객체를 만들어 낼 수 있다.

```kotlin
@Test
    fun copyTest() {
        var user = User("a","b")
        user = user.copy(surname = "c")
        println(user)
    }
    
    data class User(
        val name: String,
        val surname: String
    )
//User(name=a, surname=c)
```

## 1.2 다른 종류의 변경 가능 지점

변경할 수 있는 리스트를 만드는 방법은 두 가지가 있다.

```kotlin
//1. mutable 컬렉션 생성
val list1: MutableList<Int> = mutableListOf()
list1.add(1)
//2. var 프로퍼티 생성
var list2: List<Int> = listOf()
list2 = list2 + 1
```

두 가지 모두 += 연산자를 활용할 수 있지만 처리가 다르다. 

(참조 : [https://github.com/goyanglee/kotlin_in_action/blob/main/7. 연산자 오버로딩과 기타 관례/07.md](https://github.com/goyanglee/kotlin_in_action/blob/main/7.%20%EC%97%B0%EC%82%B0%EC%9E%90%20%EC%98%A4%EB%B2%84%EB%A1%9C%EB%94%A9%EA%B3%BC%20%EA%B8%B0%ED%83%80%20%EA%B4%80%EB%A1%80/07.md))

```kotlin
list1 += 1 //list1.plusAssign(1)
list2 += 1 // list2 = list2.plus(1)
```

첫 번째 방법은 변경 가능 지점이 구체적인 리스트 구현 내부에 있어서 멀티쓰레드 처리가 이루어질 경우 동기화 여부를 확실하게 알 수 없다.

두 번째 방법은 프로퍼티 자체가 변경 가능 지점이라 멀티쓰레드 처리가 이루어질 때 좀 더 안정성이 좋다.

두 번쨰 방법은 사용자 정의 세터나 델리게이트를 사용해서 변경을 추적할 수 있다.

```kotlin
var names by Delegates.observable(listOf<String>()) { _, old, new -> println(~)}
names += "Fabio"
names += "Bill"
```

첫 번째 방법도 추가 구현을 통해 observe할 수 있지만 두 번쨰 방법이 좀 더 사용하기 쉽고 private으로 만들수도 있다.

단, 두 방법을 모두 사용한 것은 최악의 방식이고, 모호성이 발생해 += 를 사용할 수도 없게 된다.

```kotlin
var list3 = mutableListOf<Int>()
```

## 변경 가능 지점 노출하지 말기

상태를 나타내는 Mutable 객체를 외부에 노출하는 것은 굉장히 위험하다. 예를 들어서 DB 조회해 온 mutable 객체를 외부에서 수정할 수 있게 된다.

```kotlin
data class User(val name: String)

class UserRepository {
	private val storedUsers: MutableMap<Int, String> = mutableMapOf()

	fun loadAll(): MutableMap<Int, String>) {
		return storedUsers
	}
}

val repository = UserRepository()
val storedUsers = repository.loadAll()
storedUsers[4] = "Kirill"
println(storedUsers)

//{4=Kirill}
```

해결법은 아래 두 가지가 있다.

1. 리턴되는 mutable 객체를 복제한다. 이를 방어적 복제라고 한다. (return user.copy())
2. 가변성을 제한한다. 읽기 전용 슈퍼 타입으로 업캐스트한다.

```kotlin
fun loadAll(): Map<Int, String> {
	return storedUsers
}
```

## 정리

1. var 보다는 val 을 사용하자
2. mutable 프로퍼티보다는 immutable 객체와 클래스를 사용하자
3. 변경이 필요한 대상을 만들어야 한다면 immutable 데이터 클래스로 만들고 copy 를 활용한다.
4. 컬렉션에 상태를 저장해야한다면 mutable보다는 읽기 전용 컬렉션을 사용한다.
5. 변이 지점을 적절하게 설계하고, 불필요한 변이 지점은 만들지 않는 것이 좋다.
6. mutable 객체를 외부에 노출하지 않는 것이 좋다.

가끔 효율성 때문에 immutable 보다 mutable 객체를 사용하는 것이 좋은데 성능이 중요한 부분에서만 사용하는 것이 좋다. 

immutable 객체를 사용할 때는 멀티쓰레드 사용에 더 많은 주의를 기울여야 한다.
