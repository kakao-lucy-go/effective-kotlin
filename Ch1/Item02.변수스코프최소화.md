상태를 정의할 때에는 변수와 프로퍼티의 스코프를 최소화하는 것이 좋다.

- 프로퍼티보다는 지역 변수를 사용하는 것이 좋다.
- 최대한 좁은 스코프를 갖게 변수를 사용한다. 예를 들어 반복문  안에서만 사용되는 변수라면 반복문 내부에 작성한다.

스코프는 요소를 볼 수 있는 컴퓨터 프로그램 영역으로 코틀린은 기본적으로 중괄호로 만들어지며, 내부 스코프에서 외부 스코프에 있는 요소에만 접근할 수 있다.

```kotlin
val a = 1
fun fizz() {
	val b = 2
	println(a+b)
}
val buzz = {
	val c = 3
	println(a+c)
}
//fizz 와 buzz는 a를 사용할 수 있지만 서로의 변수 b,c에는 각자 접근할 수 없다.
```

**나쁜 예**

user를 반복문 외부에서도 사용할 수 있다.

```kotlin
var user: User
for(i in users.indices) {
	user = users[i]
}
```

**조금 더 좋은 예**

```kotlin
for(i in users.indices) {
	val user = users[i]
}
```

**제일 좋은 예**

```kotlin
for((i, user) in users.withIndex()) {
}
```

스코프를 좁게 만들면 프로그램을 추적하고 관리하기 쉽다. 변경될 수 있는 부분이 많아지면 추적이 어려운데 (아이템1과도 맥락이 같다) 좁은 스코프에 걸쳐 있을 수록 변경을 추적하는 것이 쉽다.

변수는 읽기 전용 여부와 상관없이 변수를 정의할 떄 초기화되는 것이 좋다. 여러 프로퍼티를 한꺼번에 설정해야 하는 경우에는 구조분해 선언을 활용하면 좋다. (참조 [https://github.com/goyanglee/kotlin_in_action/blob/main/7. 연산자 오버로딩과 기타 관례/07.md](https://github.com/goyanglee/kotlin_in_action/blob/main/7.%20%EC%97%B0%EC%82%B0%EC%9E%90%20%EC%98%A4%EB%B2%84%EB%A1%9C%EB%94%A9%EA%B3%BC%20%EA%B8%B0%ED%83%80%20%EA%B4%80%EB%A1%80/07.md))

**나쁜 예**

```kotlin
fun updateWeather(degrees: Int) {
	val description: String
	val color: Int
	if(degrees < 5) { 
		description = "cold"
		color = Color.BLUE
	}~
}
```

**조금 더 좋은 예**

```kotlin
fun updateWeather(degrees: Int) {
	val (description, color) = when {
		degrees < 5 -> "cold" to Color.BLUE
		~
	}
}
```

## 1.1 캡쳐링

예를 들어 아래와 같은 에라토스테네스의 체 구현 코드가 있다.

```kotlin
val primes: Sequence<Int> = sequence {
	var numbers = generateSequence(2) {it +1 }
	var prime: Int
	while(true) {
		prime = numbers.first()
		yield(prime)
		numbers = numbers.drop(1)
										.filter{it % prime != 0}
	}
}
//정상 결과 : [2,3,5,7,11,13,17,19,23,29]
//이 코드 결과 : [2,3,5,6,7,8,9,10,11,12]
```

실행 결과가 이상하게 나오는데 이는 prime을 챕쳐했기 때문이다. sequence를 사용해서 필터링이 지연되어서 최종적인 prime 값만 필터링된다. prime이 2로 설정되어 있을 때 필터링된 4을 제외하면 drop만 동작하므로 그냥 연속된 숫자가 나온다.
> prime이 2가 된 후 drop 하고 filter 에 1회 실행된 후, 이걸 실질적으로 실행하는 곳(시퀀스는 최종연산때 수행이 된다. toList()처럼..) 이 없어서 filter 의 추가 수행 없이 first, drop 만 수행된다.

(시퀀스 참조 [https://github.com/goyanglee/kotlin_in_action/blob/main/5. 람다로 프로그래밍/5.md](https://github.com/goyanglee/kotlin_in_action/blob/main/5.%20%EB%9E%8C%EB%8B%A4%EB%A1%9C%20%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.md))

## 정리

변수의 스코프는 좁게 만드는 것이 좋다.
