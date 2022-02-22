### 1.5.1 코드의 동작에 제한을 거는 방법

1. require 블록: 아규먼트를 제한할 수 있다.
2. check 블록: 상태와 관련된 동작을 제한할 수 있다.
3. assert 블록: 어떤 것이 true인지 확인할 수 있다. 테스트모드에서만 동작한다.
4. return 또는 throw와 함께 활용하는 Elvis 연산자

### 1.5.2 예제

```kotlin
fun pop(num:Int = 1): List<T> {
	require(num <= size) {
		"Cannot remove more elements than current size"
	}
	check(isOpen) {"Cannot pop from closed stack"}
	val ret = collection.take(num)
	collections = collection.drop(num)
	assert(ret.size == num)
	return ret
}
```

### 1.5.3 장점

1. 문서를 읽지 않은 개발자도 문제를 확인할 수 있다.
2. 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고 예외를 throw 한다.
3. 코드가 어느 정도 자체적으로 검사돼서 이와 관련된 단위 테스트를 줄일 수 있다.
4. 스마트 캐스트 기능을 활용할 수 있게 되어 캐스트를 적게할 수 있다.

### 1.5.4 아규먼트

함수를 정의할 때 타입 시스템을 활용해서 아규먼트에 제한을 거는 코드를 사용할 수 있다.

#### 1.5.4.1 예제 

1. 숫자를 아규먼트로 받아서 팩토리얼을 계산한다면 숫자는 양의 정수여야 한다.

```kotlin
fun factorial(n: Int): Long {
	require(n >= 0)
	return if(n<=1) 1 else factorial(n-1)*n
}
```

1. 좌표들을 아규먼트로 받아서 클러스터를 찾을 때는 비어 있지 않은 좌표 목록이 필요하다.

```kotlin
fun findClusters(points: List<Point>): List<Cluster> {
	require(points.isNotEmpty())
}
```

1. 사용자로부터 이메일 주소를 입력받을 때 값이 입력되어 있는지, 이메일 형식이 올바른지 확인한다.

```kotlin
fun sendEmail(user:User, message:String) {
	requireNotNull(user.email)
	require(isValidEmail(user.email))
}
```

일반적으로 이런 제한을 걸 때에는 require 함수를 사용한다. require 함수의 조건을 만족하지 않으면 무조건 IllegalArgumentException이 발생한다. 1.5.2 예제처럼 람다로 메시지를 정의할 수도 있다.

### 1.5.5 상태

어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 할 수도 있다.

1. 어떤 객체가 미리 초기화되어 있어야만 처리를 하게 하고 싶은 함수 
2. 사용자가 로그인했을 때만 처리를 하고 싶은 함수 
3. 객체를 사용할 수 있는 시점에 사용하고 싶은 함수 

```kotlin
fun speack(text: String) {
	check(isInialized)
}
```

check 함수는 require와 비슷하지만 지정된 예측을 만족하지 못할 때 IllegalStateException을 throw한다. 상태가 올바른지 확인할 때 사용하며, require와 마찬가지로 지연 메시지를 사용해서 변경할 수 있다. 일반적으로 require 블록 뒤에 작성한다.

### 1.5.6 Assert 계열 함수 사용

확실하게 참을 낼 수 있는 코드들이 있는데 구현이 잘못됐거나 리팩토링하다가 작동하지 않을 수도 있다. 이 때 단위테스트를 사용한다.

```kotlin
@Test
fun 'Stack pops correct number of elements'() {
	val stack = Stack(20) { it }
	val ret = stack.pop(10)
	assertEquals(10, ret.size)
}

fun pop(num: Int = 1): List<T> {
	assert(ret.size == null)
	return ret
}
```

이러한 조건은 현재 코틀린/JVM에서만 활성화되며, -ea JVM 옵션을 활성화해야 확인할 수 있다. 다만 테스트할 때만 활성화되므로 check를 사용하는 것이 좋고, 단위 테스트 대신 함수에서 assert를 사용하면 아래와 같은 장점이 있다.

1. Assert계열의 함수는 코드를 자체 점검해서 효율적으로 테스트할 수 있다.
2. 특정 상황이 아닌 모든 상황에 대한 테스트를 할 수 있다.
3. 실행 시점에 정확하게 어떻게 되는지 확인할 수 있다..(?)
4. 실제 코드가 더 빠른 시점에 실패하게 만든다.

표준 애플리케이션 실행에서는 assert가 예외를 throw 하지 않고, 자바에서는 딱히 사용되지 않는다.

### 1.5.7 nullability 와 스마트 캐스팅

require와 check로 어떤 조건에 대해서 true가 나왔다면 이후로도 true라고 생각하고 스마트 캐스트가 작동한다. 아래와 같이 person.outfit이 Dress여야 코드가 정상진행이 되는데 outfit이 final 이라면(?) outfit이 Dress로 스마트캐스팅 된다.

```kotlin
fun changeDress(person: Person) {
	require(person.outfit is Dress)
	val dress: Dress = person.outfit
}
```

```kotlin
Class Person(val email: String?)

fun sendEmail(person: Person, message:String) {
	require(person.email != null)
	val email: String = person.email
}
```

reqruieNotNull, checkNotNull 이라는 특수한 함수를 사용할 수 있고, 둘 다 스마트 캐스트를 지원해서 변수를 ‘언팩'하는 용도로 활용할 수 있다.

```kotlin
fun sendEmail(person: Person, text: String) {
	val email = requireNotNull(person.email)
	validateEmail(email)
}
```

nullability 를 목적으로, 오른쪽에 throw나 return 을 두고 Elvis 연산자를 활용할 수 있다.

```kotlin
fun sendEmail(person: Person, text: String) {
	val email: String = person.email ?: return
}

fun sendEmail(person: Person, text: String) {
	val email: String = person.email ?: run {
		log(~)
	return
	}
}
```

### 1.5.8 정리

이점 

1. 제한을 훨씬 더 쉽게 확인할 수 있다.
2. 애플리케이션을 더 안정적으로 지킬 수 있다.
3. 코드를 잘 못 쓰는 상황을 막을 수 있다.
4. 스마트 캐스팅을 활용할 수 있다.

매커니즘

1. require: 아규먼트와 관련된 예측을 정의할 때 사용
2. check: 상태와 관련된 예측을 정의할 때 사용 
3. assert: 테스트코드에서 테스트를 할 때 사용
4. return 과 throw와 함께 Elvis 연산자 사용
