예를 들어 리플렉션을 사용하면, 원하는 것을 개발할 수는 있지만 private 프로퍼티와 함수의 이름 등의 세부적인 정보에 많은 의존을 하고 있기 때문에 언제든 변경할 수가 있어 프로젝트 내부에 시한 폭탄을 설치한 것과 같다.

규약을 위반하면, 코드가 작동을 멈췄을 때 문제가 된다.

## 4.32.1 상속된 규약

클래스를 상속하거나 다른 라이브러리의 인터페이스를 구현할 때에는 규약을 반드시 지켜야 한다. 예를 들어 equals와 hashCode 가 제대로 구현되지 않으면 HashSet이 제대로 동작하지 않는다.

```kotlin
class Id(val id: Int) {
	override fun equals(other: Any?) = other is Id && other.id == id
}

val set = mutableSetOf(Id(1))
set.add(Id(1))
set.add(Id(1))
print(set.size) //3
```
