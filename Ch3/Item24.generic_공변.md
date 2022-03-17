# 아이템24. 제네릭 타입과 variance 한정자를 활용하라

variance : 무공변성, 공변성, 반공변성 등에 대한 것으로, out 과 in 을 의미한다.

다음과 같은 제네릭 클래스가 있다.

```kotlin
class Cup<T>
```

T는 variance 한정자(out 또는 in)가 없어서 무공변성(invariant)이다. 예를 들어 Cup<Int>, Cup<Number>, Cup<Any>, Cup<Nothing>은 어떠한 관련성도 없다.

```kotlin
fun main() {
	val anys: Cup<Any> = Cup<Int> //type mismatch
	val nothings: Cup<Nothing> = Cup<Int>() //type mismatch
}
```

out은 T를 공변성(covariant)으로 만든다. 이는 A가 B의 서브타입일 때, Cup<A> 가 Cup<B>의 서브타입이라는 의미이다.

```kotlin
class Cup<out T>
open class Dog
class Puppy: Dog()

fun main(args: Array<String>) {
	val b: Cup<Dog> = Cup<Puppy>() //OK
	val a: Cup<Puppy> = Cup<Dog>() //type mismatch
	val anys: Cup<Any> = Cup<Int>() //OK
	val nothings: Cup<Nothing> = Cup<Int>() //type mismatch 
}
```

in은 반공변성(contravariant)으로 만든다. 이는 A가 B의 서브타입일 때, Cup<A> 가 Cup<B>의 슈퍼타입이라는 의미이다.

```kotlin
class Cup<in T>
open class Dog
class Puppy: Dog()

fun main(args: Array<String>) {
	val b: Cup<Dog> = Cup<Puppy>() //type mismatch
	val a: Cup<Puppy> = Cup<Dog>() //OK
	val anys: Cup<Any> = Cup<Int>() //type mismath
	val nothings: Cup<Nothing> = Cup<Int>() //OK
}
```

이를 그림으로 그리면 다음과 같다.

![Untitled](https://github.com/mychum1/effective-kotlin/blob/main/Ch3/effective_kotlin_01.png)

### 24.1 함수 타입

함수 타입은 파라미터 유형과 리턴 타입에 따라 서로 어떤 관계를 갖는데, 예를 들어 Int를 받고 Any를 리턴하는 함수라면 아래와 같이 쓸 수 있다.

```kotlin
fun printProcessNumber(transition: (Int) -> Any) { print(transition(42))}
```

(Int) → Number, (Number) → Any, (Number) → Number, (Number) → Int 등으로도 동작한다.

이는 코틀린 함수 타입의 모든 파라미터 타입은 반공변성이고 모든 리턴타입은 공변성이기 때문이다.

(→ Int 타입의 더 큰 타입이 들어올 수 있고, Any의 더 작은 타입이 나갈 수 있다고 이해함!)

![Untitled](https://github.com/mychum1/effective-kotlin/blob/main/Ch3/effective_kotlin_02.png)

> (?) 코틀린에서 자주 사용되는 것으로는 공변성(out)을 가진 List가 있다. variance 한정자가 붙지 않은 MutableLit와는 다르다. 왜 List를 더 많이 쓰는지 무엇이 다른지 variance 한정자의 안전성과 관련된 내용을 좀 더 살펴보자. ( 1. 그렇다고 해도 자주 사용되는 이유가 공변성이라는게 이해가 안감)
> 

```kotlin
public interface List<out E> : Collection<E> {}
```

### 24.2 variance 한정자의 안전성

자바의 배열은 공변성이다. 배열을 기반으로 정렬 함수 등을 만들기 위해서라고 하지만 큰 문제가 있다.

```java
Integer[] numbers = {1,2,3,4};
Object[] objects = numbers;
objects[2] = "B"; //ArrayStoreException: java.lang.String
```

numbers를 Objects[]로 캐스팅해도 실질적인 타입이 바뀌는게 아니라 여전히 Integer이기 때문에 오류가 발생한다. 코틀린은 이를 해결하기 위해 배열을 무공변성으로 만들었다. 따라서 Array<Int> 를 Array<Any>등으로 바꿀 수 없다.

```kotlin
open class Dog
class Puppy: Dog()
class Hound: Dog()

fun takeDog(dog: Dog) {}

takeDog(Dog())
takeDog(Puppy())
takeDOg(Hound())
```

> (?) 파라미터 타입을 예측할 수 있다면 암묵적으로 업캐스팅할 수 있는데 이는 공변적이지 않다.(Dog 타입에 Dog를 확장한 Puppy와 Hound가 들어가면서 타입이 업!) out 한정자가 in 한정자 위치(예를 들어 파라미터 타입)에 있다면 업캐스팅 + 공변적으로 돼서(서브 타입이 들어갈 수 있다) 우리가 원하는 타입을 아무 것이나 전달할 수 있다(?). 즉, value가 매우 구체적인 타입이라 안전하지 않으므로(?), value를 Dog 타입으로 지정할 경우, String 타입을 넣을 수 없다.
> 

```kotlin
class Box<out T> {
    private var value: T? = null
    
    //사용불가 코드
    fun set(value: T) { //Type parameter T is declared as 'out' but occurs in 'in' position in type T
        this.value = value
    }
    
    fun get(): T = value ?: error("Value not set")
}

val puppyBox = Box<Puppy>()
val dogBox: Box<Dog> = puppyBox
dogBox.set(Hound()) //Puppy로 T가 구체화되었기 때문에 Hound가 들어갈 수 없다.

val dogHouse = Box<Dog>()
val box: Box<Any> = dogBox
box.set("Some string") //Dog로 구체화되었기 떄문에 문자열이 들어갈 수 없다.
```

캐스팅 후에 실질적인 객체가 그대로 유지되기 때문에 안전하지 않다.(유지가 되지 않다면 Any 타입 box에 문자열, Int 전부 다 들어올 수 있었을 것.) 그래서 코틀린은 public In 한정자 위치에 반공변성 타입 out 한정자가 오는 것을 금지한다.

```kotlin
class Box<out T> {
    var value: T? = null //public in 위치. 오류 Type parameter T is declared as 'out' but occurs in 'invariant' position in type T?
    private var value: T? = null //private in 위치. 오류나지 않음
    //사용불가 코드
    fun set(value: T) { //Type parameter T is declared as 'out' but occurs in 'in' position in type T
        this.value = value
    }

    fun get(): T = value ?: error("Value not set")
}
```

private으로 가시성을 제한하면 객체 내부에서는 업캐스트 객체에 out을 사용할 수 없기 때문에 오류가 발생하지 않는다. (바로 접근해서 가져갈 수 없고 다른 게터를 써서 가져가야 하니까 out을 사용할 수 없다.)

> (?) out 한정자는 public out 한정자 위치에서도 안전하므로..? 따로 제한하지 않는다. 이러한 안정성의 이유로 생성되거나 노출되는 타입에만 out 한정자를 사용한다. 이러한 프로퍼티는 일반적으로 producer 또는 immutable 데이터 홀더에 많이 사용된다. (데이터 홀더가 뭐야?)
> 

> 좋은 예로, T는 공변적인 List<T>가 있다. List<Any?>로 예측된다면 별도의 변환 없이 모든 종류를 파라미터로 전달할 수 있다. 하지만 MutableList<T> 는 in 한정자 위치에서만 사용되고 안전하지 않으므로 무공변적이다.(?)(하지만 아래 구현속 T는 둘다 같은걸?)
> 

```kotlin
val strs = mutableListOf<String>("a")
append(strs) // < error
val str: String = strs[1]
print(str)

//Type mismatch.
//Required:
//MutableList<Any>
//Found:
//MutableList<String>
fun append(list: MutableList<Any>) {
	list.add(42)
}
```

```kotlin
@kotlin.SinceKotlin @kotlin.internal.InlineOnly public inline fun <T> List(size: kotlin.Int, init: (kotlin.Int) -> T): kotlin.collections.List<T> { /* compiled code */ }

@kotlin.SinceKotlin @kotlin.internal.InlineOnly public inline fun <T> MutableList(size: kotlin.Int, init: (kotlin.Int) -> T): kotlin.collections.MutableList<T> { /* compiled code */ }
```

또 다른 좋은 예로는 Response가 있다. variance 한정자 덕분에 이 내용은 모두 참이 된다.

1. Response<T> 라면 T의 모든 서브 타입이 허용된다. 예를 들어 Response<Any> 가 예상된다면, Response<Int>와 Response<String>이 허용된다.
2. Response<T1, T2> 라면 T1과 T2의 모든 서브타입이 허용된다.
3. Failure<T>라면 T의 모든 서브타입 Failure가 허용된다. 예를 들어 Failure<Number> 라면, Failure<Int> 와 Failure<Double> 이 모두 허용된다.
4. 다음 코드 처럼 convariant(공변) 와 Nothing 타입으로 인해 Failure는 오류 타입을 지정하지 않아도 되고, Success는 잠재적인 값을 지정하지 않아도 된다.

```kotlin
sealed class Response<out R, out E> //R: 값, E: 오류 타입
class Failure<out E>(val error: E): Response<Nothing, E>()
class Success<out R>(val value: R): Response<R, Nothing>()
```

> convariant와 public in 위치와 같은 문제는 contravariant(반공변성) 타입 파라미터 in과 public out 위치(함수 리턴 타입 또는 프로퍼티 타입)에서도 발생한다. out 위치는 암묵적 업캐스팅을 허용한다.(out 이 어딨어?)
> 

```kotlin
open class Car
interface Boat
class Amphibious: Car(), Boat

fun getAmphibious(): Amphibious = Amphibious()

val car: Car = getAmphibious()
val boat: Boat = getAmphibious()
```

이는 contravariant(반공변. in 한정자) 에 맞는 동작은 아니다. 다음 코드는 Box에 어떤 타입이 들어있는 지 확실하게 알 수 없다.

```kotlin
class Box<in T> {
	val value: T //Type parameter T is declared as 'in' but occurs in 'out' position in type T
}

val garage: Box<Car> = Box(Car())
val amphibiousSpot: Box<Amphibious> = garage
val boat: Boat = garage.value //
//Type mismatch.
//        Required:
//        Boat
//        Found:
//        Car

val noSpot: Box<Nothing> = Box<Car>(Car())
val boat: Nothing = noSpot.value //Conflicting declarations: val boat: Boat, val boat: Nothing
```

코틀린은 이때문에 contravariant 타입 파라미터(in 한정자)를 public out 한정자 위치에 사용하는 것을 금지하고 있다.(근데 위에 예제에서 부터 코드가 비슷하고 in, out만 바꼈는데 대체 뭘 금지하는건지 모르겠다..)

```kotlin
class Box<in T> {
    var value: T? = null //Type parameter T is declared as 'in' but occurs in 'invariant' position in type T?

    fun set(value: T) {
        this.value = value

    }

    fun get(): T = value ?: error("value not set") //Type parameter T is declared as 'in' but occurs in 'out' position in type T
}
```

private 으로 가시성을 제한하면 된다.

```kotlin
class Box<in T> {
    private var value: T? = null

    fun set(value: T) {
        this.value = value

    }

    private fun get(): T = value ?: error("value not set")
}
```

타입 파라미터에 contravariant(in 한정자)를 사용하는게 많이 사용되는 예로는 kotlin.coroutines.Continutation이 있다.

### 24.3 variance 한정자의 위치

variance 한정자는 크게 두 위치에서 사용할 수 있다.

1. 선언 부분 : 일반적. 클래스와 인터페이스가 사용되는 모든 곳에 영향을 준다.

```kotlin
class Box<out T>(val value: T)
val boxStr: Box<String> = Box("Str")
val boxAny: Box<Any> = boxStr
```

1. 클래스와 인터페이스를 활용하는 위치

이 위치에 사용하면 특정한 변수에만 variance 한정자가 적용된다.

```kotlin
class Box<T>(val value: T) {
	val boxStr: Box<String> = Box("Str")
//사용하는 쪽의 variance 한정자 
	val boxAny: Box<out Any> = boxStr
}
```

모든 인스턴스가 아니라 특정 인스턴스에만 적용해야할 때 이런 코드를 사용한다. 예를 들어 MutableList는 in 한정자를 포함하면 요소를 리턴할 수 없기 때문에 in 한정자를 붙이지 않는다. 하지만 단일 파라미터 타입에 in 한정자를 붙여서 contravariant를 가지게 해서 여러 타입을 받아들일 수 있게 한다.

```kotlin
interface Dog
interface Cutie
data class Puppy(val name: String): Dog, Cutie
data class Hound(val name: String): Dog

fun fillWithPuppies(list: MutableList< in Puppy>) {
	list.add(Puppy("Jim"))
list.add(Puppy("Bim"))
}

val dogs = mutableListOf<Dog>(Hound("Pluto"))
fillWithPuppies(dogs)
println(dogs) //[Hound(name=Pluto), Puppy(name=Jim), Puppy(name=Bim)]
```

> variance 한정자를 사용하면 위치가 제한될 수 있다. 예를 들어 MutableList<out T> 가 있다면, get으로 요소를 추출했을 때 T타입이 나올 것 이지만 set은 Nothing 타입이 들어올 거라 예상되므로 사용할 수 없다.이는 모든 타입의 서브타입을 가진 Nothing 리스트가 존재할 수 있기 때문이다.(Nothing 타입이 들어올거라 예상하는게 아니라 Nothing 도 들어올 수 있기 때문에 안된단건가?)
> 

> MutableList<in T>를 사용한다면 get, set을 모두 사용할 수 있는데 get은 Any?가 전달된다. 이는 모든 타입의 슈퍼타입을 가진 Any 리스트가 존재할 수 있기 때문이다.(왜 Any? 가 전달되지? T가 Number로 들어온다고 해도 Any?가 전달된다는건가? Any?가 전달될 수 있다는 이야기)
> 

### 24.4 정리

코틀린은 다음과 같은 타입 한정자가 있다.

- 타입 파라미터의 기본적인 variance의 동작은 invariant(무공변) 이다. 만약 Cup<T> 라면 T는 invariant이다. A가 B의 서브타입일 때 Cup<A> 와 Cup<B>는 아무 관계가 없다.
- out 한정자는 타입 파라미터를 convariant하게 만든다. 만약 Cup<out T>라고 하면 T는 covariant이고, A가 B의 서브타입일 때 Cup<A> 는 Cup<B>의 서브타입이다. (Cup<T>라고 되어있는데.. out 이 써져야 하는거 아닌가?
- in 한정자는 타입 파라미터를 contravariant하게 만든다. 만약 Cup<in T> 라고 한다면 T는 contravariant이다. A가 B의 서브타입일 때 Cup<B>는 Cup<A>의 슈퍼타입이다. (Cup<T>라고 되어있는데.. in 이 써져야 하는거 아닌가?

코틀린은,

- List와 Set의 타입 파라미터는 covariant(out) 이다.

> List<Any>가 예상되는 모든 곳에 전달할 수 있다.
> 

Map에서 값의 타입을 나타내는 T는 covariant(out)이다. Array, MutableList, MutableSet, MutableMap의 T는 invariant이다.

- 함수 타입의 T는 contravariant(in)이고, 리턴 타입은 covariant(out)이다. (리턴타입은 contravariant라고 써있는데.. covariant아닌가?)
- 리턴만 되는 타입에는 covariant(out) 한정자를 사용한다.
- 허용만 되는 타입에는 contravariant(in) 한정자를 사용한다.
