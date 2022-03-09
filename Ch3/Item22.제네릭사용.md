# 아이템22.일반적인 알고리즘을 구현할 때 제네릭을 사용하라

타입 아규먼트를 사용하는 함수를 제네릭 함수라고 하고 대표적으로 stdlib의 filter가 있다.

```kotlin
inline fun <T> Iterable<T>.filter (
	preficate: (T) -> Boolean
): List<T> {
~
}

```

타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공하여 컴파일러가 더 정확하게 타입을 추측할 수 있도록 해준다. 컴파일 과정에서 최종적으로 타입 정보는 사라지지만 개발 중에는 특정 타입을 사용하도록 강제할 수 있어서 안전하게 사용할 수 있다. Set<User>에서 데이터를 꺼내면 User 타입이라는 것을 알 수 있다. 

## 22.1 제네릭 제한

타입 파라미터의 중요한 기능 중 하나는 구체적인 타입의 서브타입만 사용하게 타입을 제한하는 것이다. 예를 들어 Iterable<Int>의 서브타입으로 제한하면 T 타입을 기반으로 반복 처리가 가능하고, 이 때 사용하는 객체가 Int라는 것을 알 수 있다. 

```kotlin
fun <T: Comparable<T>> Iterable<T>.sorted(): List<T> {~}
```

많이 사용하는 제한으로는 Any가 있다. 이는 nullable이 아닌 것을 의미한다. 

```kotlin
inline fun<T, R: ANy> Iterable<T>.mapNotNull(
	transform: (T) -> R?
): List<R> {
	return mapNotNullTO(ArrayList<R>(), transform)
}
```

드물지만 둘 이상의 제한을 걸 수도 있다.
