# 아이템33. 생성자 대신 팩토리 함수를 사용하라

클래스의 인스턴스를 만드는 가장 일반적인 방법은 기본 생성자를 사용하는 것이다. 다른 방법으로는 디자인 패턴으로 다양한 생성 패턴들이 만들어져 있는데 보통 객체를 생성자로 직접 생성하지 않고 별도의 함수를 통해 생성하는 것이다.

아래 코드는 MyLinkedList 클래스의 인스턴스를 만들어서 제공하는 톱레벨 함수이다.

```kotlin
fun <T> myLinkedListOf(
	vararg elements: T
) : MyLinkedList<T>? {
	if(elements.isEmpty()) return null
	val head = elements.first()
	val elementsTail = elements.copyOfRange(1, elements.size)
	val tail = myLinkedListOf(*elemtnsTail)
	return MyLinkedList(head, tail)
}

val list = myLinkedListOf(1,2)
```

이처럼 생성자의 역할을 대신 해주는 함수를 **팩토리 함수**라고 한다.

### 5.33.1. 팩토리 함수의 장점

1. 생성자와는 달리 함수에 이름을 붙여서 객체가 생성되는 방법, 무슨 아규먼트가 필요한 지 설명할 수 있다. ArrayList(3) 보다는 ArrayList.withSize(3) 이 더 이해하기가 쉽다. 또, 동일한 파라미터 타입을 갖는 생성자와의 충돌을 방지할 수 있다.
2. 생상자와는 달리 원하느 형태의 타입을 리턴할 수 있다. 인터페이스 뒤에 실제 객체의 구현을 숨길 때 사용할 수 있다. 예를 들어 listOf는 List 인터페이스를 리턴하는데 플랫폼에 따라 다른 객체를 리턴한다.
3. 생성자와는 달리 호출할 때마다 새로운 객체를 만들 필요가 없다. 싱글톤 패턴처럼 하나만 생성하게 할 수 있고, 캐싱 메커니즘을 사용할 수 있으며 null 을 리턴할 수도 있다. Connection.createOrNull 과 같이 연결을 할 수 없을 때 null 을 리턴한다.
4. 아직 존재하지 않는 객체를 리턴할 수 있다. 그래서 어노테이션 처리를 기반으로 하는 라이브러리에서는 팩토리 함수를 많이 사용한다. 프로젝트를 빌드하지 않아도 앞으로 만들어질 객체를 사용하거나 프록시를 통해 만들어지는 객체를 사용할 수도 있다.(?)[https://hinos.tistory.com/131](https://hinos.tistory.com/131) 
5. 객체 외부에 팩토리 함수를 만들어서 가시성을 제어한다. 예를 들어 톱레벨 팩토리 함수를 같은 파일이나 같은 모듈에서만 접근하게 할 수 있다.
6. 팩토리 함수는 인라인으로 만들 수 있고, 파라미터는 reified로 만들 수 있다.
7. 생성자로 만들기 복잡한 객체도 만들 수 있다.
8. 생성자는 슈퍼클래스나 기본 생성자를 호출해야 하지만, 팩토리 함수는 원하는 때에 생성자를 호출할 수 있다.

### 5.33.2. 팩토리 함수의 제한

1. 팩토리 함수로 클래스를 생성할 때 서브 클래스 생성에는 슈퍼 클래스의 생성자가 필요하기 때문에 서브클래스를 만들 수 없다. (다만이라고 묶을 건 아니고, 위의 장점 8번에 대한 추가 설명인 것 같은데..)

```kotlin
class IntLinkedList: MyLinkedList<Int>() {
	constructor(vararg ints: Int): myLinkedListOf(*ints) //오류
}
```

```kotlin
class MyLinkedIntList(head: Int, tail: MyLinkedIntList?): MyLinkedList<Int>(head, tail)

fun myLinkedIntListOf(vararg elements: Int): MyLinkedIntList {
	if(elements.isEmpty()) return null 
	val head = elements.first()
	val elementsTail = elements.copyOfRange(1, elements.size)
	val tail = myLinkedIntListOf(*elementsTail)
	return MyLinkedIntList(head, tail)
}
```

팩토리 함수가 강력하긴 하지만 기본 생성자를 쓰지 말라는 이야기는 아니다. 팩토리 함수 내부에서는 생성자를 사용해야 한다. 팩토리 함수는 기본 생성자가 아니라 추가 생성자와 경쟁적인 관계이고, 추가 생성자보다는 팩토리 함수를 많이 사용한다.

팩토리 함수에는 다음과 같이 5가지 종류가 있다.

1. companion 객체 팩토리 함수
2. 확장 팩토리 함수 
3. 톱레벨 팩토리 함수 
4. 가짜 생성자
5. 팩토리 클래스의 메소드

### 5.33.3. Companion 객체 팩토리 함수

```kotlin
class MyLinkedList<T> (val head: T, val tail: MyLinkedList<T>?){
	companion object{
		fun <T> of(vararg elements: T): MyLinkedList<T>? {~}
	}
}

val list = MyLinkedList.of(1, 2)
```

이를 이름을 가진 생성자(Named Constructor Idiom) 이라고 부르며, 생성자와 같은 역항르 하면서 다른 이름이 있는 것을 의미한다.

코틀린에서는 이를 인터페이스에도 구현할 수 있다.

```kotlin
class MyLinkedList<T> (val head: T, val tail: MyLinkedList<T>?): MyList<T> {~}

interface MyList<T> {
	companion object {
		fun <T> of(vararg elements: T): MyList<T>?{~}
	}
}
```

이 외에도 다음과 같은 이름이 사용된다.

1. from: 파라미터를 하나 받고 같은 타입의 인스턴스 하나를 반환 

```kotlin
val date: Date = Date.from(instant)
```

1. of: 파라미터를 여러개 받고 통합해서 인스턴스 생성 

```kotlin
val faceCards: Set<Rank> = EnumSet.of(JACK, QUEEN, KING)
```

1. valueOf: from 이나 of 와 비슷한데 좀 더 쉽게 읽을 수 있다.

```kotlin
val prime: BigInteger = BigInteger.valueOf(Integer.MAX_VALUE)
```

1. instance 또는 getInstance: 싱글톤으로 인스턴스 하나를 리턴. 파라미터가 있으면 이를 기반으로 만든다. 같은 파라미터면 같은 인스턴스를 반환한다.

```kotlin
val luke: StackWalker = StackWalker.getInstance(options)
```

1. createInstance 또는 newInstance: getInstance처럼 동작하지만 싱글톤은 아니라서 함수를 호출할 때마다 인스턴스 반환 

```kotlin
val newArray = Array.newInstance(classObject, arrayLen)
```

1. getType: getInstance처럼 동작하지만 팩토리 함수가 다른 클래스에 있을 때 사용한다.

```kotlin
val fs: FileStore = Files.getFileStore(path)
```

1. newType: newIntance처럼 동작하지만, 팩토리 함수가 다른 클래스에 있을 때 사용한다.

```kotlin
val br: BufferedReader = Files.newBufferedReader(path)
```

companion 객체 멤버는 단순 정적 멤버처럼 다루는 것 말고도 인터페이스를 구현할 수 있고, 클래스를 상속받을 수도 있다. 보통 다음과 같은 형태로 팩토리 함수를 만든다.

```kotlin
abstract class ActivityFactory {
	abstract fun getIntent(context: Context): Intent
	fun start(context: Context) {
		val intent = getIntent(context)
		context.startActivity(intent)
	}

	fun startForResult(activity: Activity, requestCode: Int) {
		val intent = getIntent(activity)
		activity.startActivityForResult(intent, requestCode)
	}
}

class MainActivity: AppCompatActivity() {
	companion object: ActivityFactory() { //ActivityFactory를 상속했다.
		override fun getIntent(context: Context): Intent = Intent(context, MainActivity::class.java)
	}
}

val intent = MainActivity.getIntent(context)
MainActivity.start(context) //이게 아닌 것 같은데
MainActivity.startForResult(activity, requestCode)

intent.start(context) //이것인듯
intent.startForResult(activity, requestCode)
```

추상 companion 객체 팩토리는 값을 가질 수 있어서 캐싱을 구현하거나 가짜 객체를 생성할 수도 있다. 

코틀린 코루틴 라이브러리를 보면 거의 모든 코루틴 컨텍스트의 companion 객체가 컨텍스트를 구별할 목적으로 CoroutineContext.Key 인터페이스를 구현하고 있다.

### 5.33.4. 확장 팩토리 함수

사용하려는 인터페이스나 클래스에 companion object가 존재할 때 팩토리함수를 사용해야할 때가 있다. 직접 companion object를 수정하지 않고도 만들어야 한다면 확장함수를 사용하면 된다.

```kotlin
interface Tool {
	companion object {~}
}

fun Tool.Companion.createBigTool(~) : BigTool {~}
```

이런 식으로 사용하면 외부 라이브러리를 확장할 수도 있는데, 단, 적어도 비어있는 companion object가 필요하다.

### 5.33.5. 톱레벨 팩토리 함수

대표적인 예로 listOf, setOf, mapOp가 있다. List.of(1,2,3) 보다 listOf(1,2,3)이 훨씬 읽기 쉽기 때문에 톱레벨 함수를 사용한다. 하지만 public 톱레벨 함수는 모든 곳에서 사용할 수 있기 때문에 IDE가 제공하는 팁을 복잡하게 만들 수도 있고 클래스 메소드처럼 이름을 만들면 혼란을 일으킬 수 있다.

### 5.33.6. 가짜 생성자

코틀린의 생성자는 톱레벨 함수와 같은 형태로 사용되기 때문에 톱레벨 함수처럼 참조될 수 있다. (생성자 레퍼런스는 함수 인터페이스로 구현한다.(?))

```kotlin
class A
val a = A()
val reference: () -> A ::A
```

일반적인 관점에서 생성자는 대문자로 시작하고 함수는 소문자로 시작한다. 하지만 함수가 대문자로 사용할 수도 있는데, 예를 들어 List와 MutableList는 인터페이스라 생성자를 가질 수 없지만 List를 생성자처럼 사용하는 코드가 있다.

```kotlin
List(4) {'User$it }
```

바로 List라는 함수가 코틀린 1.1부터 stdlib에 포함되었기 때문이다.

```kotlin
public inline fun <T> List(size: Int, init: (index: Int) -> T): List<T> = MutableList(size, init)
public inline fun <T> MutableList(size: Int, init: (index:Int) -> T): MutableList<T> { 
	val list = ArrayList<T>(size)
	repeat(size) {index -> list.add(init(index)) }
	return list
}
```

이런 톱레벨 함수는 생성자처럼 보이고 생성자처럼 작동하며 팩토리 함수와 같은 모든 장점을 갖는다. 이를 **가짜 생성자(fake constructor)** 라고 한다.

- 인터페이스를 위한 생성자를 만들고 싶을 때
- reified 타입 아규먼트를 갖고 싶을 때

가짜 생성자를 사용한다. (이를 제외하면??(?)) 가짜 생성자는 진짜 생성자처럼 동작하며 생성자처럼 보여야한다. 캐싱이나 nullable 타입 리턴, 서브 클래스 리턴 등의 기능을 포함하고 싶으면 companion object에 팩토리 함수를 사용하는 것이 좋다.

가짜 생성자를 선언할 때 invoke 연산자를 갖는 compnanion object를 사용할 수도 있지만 잘 사용하진 않고 ‘아이템 12’에서 살펴본 것처럼 연산자 오버로드는 의미에 맞게 하라는 규칙에 위배되기 때문에 추천하지 않는다.

invoke는 이름 없이 호출될 수 있는 특별한 함수이기 때문에 객체 생성과는 의미가 다르기 때문이다.

```kotlin
class Tree<T> {
	companion object{
		operator fun <T> invoke(size: Int, generator: (Int) -> T): Tree<T> {~}
	}
}

Tree(10) {"$it")
```

또, 지금까지 살펴본 방법을들 모아서 보면 Invoke 함수가 훨씬 복잡하다는 것을 알 수 있다.

1. 생성자

```kotlin
val f: () -> Tree = ::Tree
```

1. 가짜 생성자

```kotlin
val f: () -> Tree = ::Tree
```

1. invoke 함수를 가지는 companion 객체

```kotlin
val f: () -> Tree = Tree.companion::invoke
```

가짜 생성자는 톱레벨 함수를 사용하는 것이 좋고, 기본 생성자를 만들 수 없거나 생성자가 제공하지 않는 기능(reified 타입 파라미터 등: reified가 뭐였지 )으로 생성자를 만들어야 하는 경우에 가짜 생성자를 만드는 것이 좋다.

### 5.33.7. 팩토리 클래스의 메소드

코틀린에서 점층적 생성자 패턴과 빌더 패턴은 코틀린에서는 의미가 없다.(아이템 34) 팩토리 클래스는 클래스의 상태를 가질 수 있다는 특징 때문에 팩토리 함수보다 다양한 기능을 갖는다.

```kotlin
data class Student(val id: Int, val name: String, val surname: String)

class StudentFactory {
	var nextId = 0
	fun next(name: String, surname: String) = Student(nextId++, name, surname)
}

val factory = StudentFactory()
val s1 = factory.next("Marcin", "Moskala")
```

팩토리 클래스는 프로퍼티를 가질 수 있어서 캐싱을 활용하거나 이전에 만든 객체를 복제해서 객체를 생성하는 등의 기능을 도입할 수 있다.

### 5.33.8. 정리

1. 가짜 생성자
2. 톱레벨 팩토리 함수
3. 확장 팩토리 함수 - companion object 사용 

3번이 가장 자바 정적 팩토리 매소드 패턴과 유사해서 일반적으로 사용하는 패턴이다.
