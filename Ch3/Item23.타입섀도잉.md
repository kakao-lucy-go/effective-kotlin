# 아이템23.타입 파라미터의 섀도잉을 피하라

프로퍼티와 파라미터가 같은 이름을 가질 수 있다. 이러면 지역 파라미터가 외부 스코프에 있는 프로퍼티를 가리게 되는데, 이를 섀도잉(shadowing)이라고 한다.

```kotlin
class Forest(val name: String) {
	fun addTree(name: String) {~}
}
```

이런 현상은 클래스 타입 파라미터와함수 타입 파라미터 사이에도 발생한다. 제네릭을 제대로 이해하지 않고 사용하면 그때는 스스로 문제를 찾기 어려울 수있다.

```kotlin
interface Tree
class Birch: Tree
class Spruce: Tree

class Forest<T: Tree> {
	fun <T: Tree> addTree(tree: T) {~}
}
```

위와 같은 코드는 Forest와 addTree의 타입 파라미터가 독립적으로 동작한다.

```kotlin
val forest = Forest<Birch>()
forest.addTree(Birch())
forest.addTree(Spruce())
```

따라서 addTreer가 클래스 타입 파라미터인 T를 사용하게 하는 것이 좋다.

```kotlin
interface Tree
class Birch: Tree
class Spruce: Tree

class Forest<T: Tree> {
	fun addTree(tree: T) {~}
}
```

독립적으로 사용하는 것이라면 이름을 아예 다르게 다는 것이 좋다. 

다음처럼 타입 파라미터를 사용해서 다른 타입 파라미터에 제한을 줄 수 있다.

```kotlin
class Forest<T: Tree> {
	fun <ST: T> addTree(tree: ST)
}
```

결론적으로 섀도잉을 피하는 것이 좋다.
