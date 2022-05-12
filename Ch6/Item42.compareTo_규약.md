# 아이템 42. compareTo의 규약을 지켜라

compareTo는 Any에 있는 메소드가 아니라 부등식으로 변환되는 연산자이다.

1. obj1 > obj2 //obj1.compareTo(obj2) > 0
2. obj1 < obj2 //obj1.compareTo(obj2) < 0
3. obj1 ≥ obj2 // obj1.compareTo(obj2) ≥ 0
4. obj1 ≤ obj2 // obj1.compareTo(obj2) ≤ 0 

(이러면, obj1 - obj2 를 계산한다는 의미가 되는건가?)

compareTo는 어떤 순서를 가지고 비교할 수 있다는 것을 의미한다. compareTo는 다음과 같이 동작해야 한다.

1. 비대칭적 동작: a≥b 이고 b≥a 라면 a==b이다.
2. 연속적 동작: a≥b 이고 b≥c라면 a≥c이다.
3. 코넥스적 동작(connex relation): a≥b 또는 b≥a 중에 적어도 하나는 반드시 항상 true여야 한다. 관계가 없다면 고전적 정렬 알고리즘(퀵, 삽입)을 사용할 수 없고, 위상 정렬만 사용할 수 있다.

## compareTo를 따로 정의해야 할까?

일반적으로는 없다. sortedBy(단일 키), sortedWith(여러 키)로 정렬할 수 있기 때문이다. 

물론 Comparable<T>로 원하는 형태로 정렬하게 할 수 있다. 문자열은 알파벳과 숫자등의 순서가 있고 내부적으로 Comparable<String>을 구현하고 있어서 매우 유용하다. 하지만 알파벳 부등호는 가독성이 좋지 않아서 권장하진 않는다.

```kotlin
println("Kotlin" > "Java")//true
```

## compareTo 구현하기

톱레벨 함수를 활용할 수 있다.

```kotlin
class User(
	val name: String,
	val surname: String
): Comparable<User> {
	override fun compareTo(other: User): Int = compareValus(surname, other.surname)
}
```

compareValuesBy로 여러 프로퍼티를 비교할 수 있다.

비교할 때에는 반환되는 값을 잘 고려해야 한다.

- 0: 리시버와 other가 같은 경우
- 양수: 리시버가 other보다 큰 경우
- 음수: 리시버가 other보다 작은 경우
