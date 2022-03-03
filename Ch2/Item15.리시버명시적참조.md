명시적으로 긴 코드를 사용할 때가 있다. 예를 들어서 함수와 프로퍼티를 다른 리시버로부터 가져온다는 것을 나타낼 때 this를 쓰는 것 등이 있다.

```kotlin
class User:Person() {
	private var beersDrunk: Int = 0
	fun drinkBeers(num: Int) {
		this.beersDrunk += num
	}
}
```

## 15.1 여러 개의 리시버

스코프 내부에 둘 이상의 리시버가 있는 경우 명시적으로 나타내면 좋다. apply, with, run 함수를 사용할 때가 대표적인 예다.

```kotlin
class Node(val name: String) {
	fun makeChild(childName: String) = create("$name.$childName")
																			.apply { print("Created ${this?.name}") } //Node unpack

	fun create(name: String) : Node? = Node(name)
}

fun main() {
	val node = Node("parent")
	node.makeChild("child") //Created parent.child
}
```

사실 이는 apply의 잘못된 사용 예제다. also를 사용하면 이전과 마찬가지로 명시적으로 리시버를 지정할 수 있다. also나 let을 사용해서 nullable을 처리할 수 있다.

1. 리시버가 명확하지 않다면 명시적으로 적는다.
2. 레이블 없이 리시버를 사용하면 가장 가까운 리시버를 의미한다.
3. 외부에 있는 리시버를 사용하려면 레이블을 사용한다.

```kotlin
class Node(val name: String) {
	fun makeChild(childName: String) = 
		create("$name.$.childName").apply {
			print("Created ${this?.name} in ${this@Node.name}") //2번 3번
		}
}
```

## 15.2 DSL 마커

코틀린 DSL을 사용할 때에는 리시버들이 중첩되더라도 명시적으로 붙이지 않고 사용하게 설계되어 있지만, 외부의 함수를 사용하는 것이 위험한 경우가 있다. 사용하지 않으려면 DslMarker로 암묵적으로 외부 리시버를 막거나 명시적으로 라벨링해서 사용해야 한다.

## 15.3 정리

1. 여러 리시버가 있다면 명시적으로 적어준다
2. 짧게 적을 수 있다고 리시버를 제거하지 않는다.
