```kotlin
operator fun Int.not() = factorial()
println(10 * !6)
```

팩토리얼이 ! 기호를 쓴다고 해서 의미에 맞지 않은 연산자를 오버로딩해서 쓰면 안된다.

| 연산자 | 대응되는 함수 |
| --- | --- |
| +a | a.unaryPlus() |
| -a | a.unaryMinus() |
| !a | a.not() |
| ++a | a.inc() |
| —a | a.dec() |
| a+b | a.plus(b) |
| a-b | a.minus(b) |
| a*b | a.times(b) |
| a/b | a.div(b) |
| a..b | a.rangeTo(b) |
| a in b  | b.contains(a) |
| a+=b | a.plusAssign(b) |
| a-=b | a.minusAssign(b) |
| a*=b | a.timesAssign(b) |
| a/=b | a.divAssign(b) |
| a==b | a.equals(b) |
| a>b | a.compareTo(b) > 0 |
| a<b | a.compareTo(b) < 0 |
| a≥b | a.compareTo(b) ≥ 0 |
| a≤b | a.compareTo(b) ≤ 0 |

## 12.1 분명하지 않은 경우

하지만 관례를 충족하는지 확실하지 않을 때 의미가 명확하지 않다면 infix 를 활용한 확장 함수를 사용하는 것이 좋다.

```kotlin
infix fun Int.timesRepeated(operation: () -> Unit) = {
	repeat(this) {operation() }
}

val tripleHello = 3 timesRepeated {print("Hello")}
tripleHello()
```

혹은 톱레벨 함수를 사용하는 것도 좋다.

## 12.2 규칙을 무시해도 되는 경우

연산자 오버로딩을 무시해도 되는 경우는 도메인 특화 언어(DSL: Domain Specific Language) 를 설계할 때이다. 

## 12.3 정리

1. 연산자 오버로딩은 그 이름의 의미에 맞게 사용한다.
2. 연산자 같은 형태로 사용하고 싶다면 infix 확장함수 또는 톱레벨 함수를 활용한다.
