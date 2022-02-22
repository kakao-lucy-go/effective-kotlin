타입 추론(type inference)은 피연산자의 타입에 맞게 설정이 되고, 절대로 슈퍼클래스나 인터페이스로는 설정되지 않는다.

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
	var animal = Zebra()
	animal = Animal() // type mismatch
}
```

타입을 명시적으로 지정하면 이 문제를 해결할 수 있다.

```kotlin
var animal: Animal = Zebra()
```

하지만 라이브버리나 모듈을 직접 조작할 수 없는 경우에는 문제가 발생할 수 있다.

```kotlin
//1차
interface CarFactory {
	fun produce(): Car
}

val DEFAULT_CAR: Car = Fiat126P() //대부분 Fiat126P를 생산하기 때문에 기본값으로 설정

//2차. DEFAULT_CAR가 Car로 명시적으로 지정되어 있으므로 따로 필요 없다고 생각해서 리턴타입 제거
interface CarFactory {
	fun produce() = DEFAULT_CAR
}

//3차. CarFactory에서는 이제 Fiat126P 이외의 자동차는 생상하지 못한다.
val DEFAULT_CAR = Fiat126P()

//> 결과
interface CarFactory {
	fun produce() = DEFAULT_CAR
}
val DEFAULT_CAR = Fiat126P()
```

타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정해야한다는 원칙을 갖고 있어야 한다.

외부 API를 만들 때에는 반드시 타입을 지정하고 특별한 이유나 확실한 확인 없이 제거하지 않는다.
