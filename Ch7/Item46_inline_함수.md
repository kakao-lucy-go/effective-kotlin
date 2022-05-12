# 아이템 46. 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라

대부분 고차함수에는 inline 한정자가 붙어있다. 

```kotlin
inline fun repeat(times: Int, action: (Int) -> Unit) {
	for(index in 0 until times) { action(index) }
}
```

inline은 컴파일 시점에 ‘함수를 호출하는 부분'을 ‘함수의 본문'으로 대체하는 것이다.

```kotlin
repeat(10) {println(it)}

>

for(index in 0 until 10) { println(index) }
```

함수를 호출하는 부분이 없어지고 바로 본문이 실행된다.

**장점**

1. 타입 아규먼트에 reified 한정자를 붙여서 사용할 수 있다.
- reified 는 제네릭을 런타임에 타입 정보를 알고 싶을 때 사용
    
    
1. 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다. → 호출하는 과정이 없기 때문에
2. non-local 리턴을 사용할 수 있다. 
- non-local return: 인라인 함수에 사용된 람다의 return은 람다를 인자로 받는 인라인 함수도 함께 종료시킨다. [https://blog.naver.com/PostView.nhn?blogId=yuyyulee&logNo=221390355858](https://blog.naver.com/PostView.nhn?blogId=yuyyulee&logNo=221390355858)
    
    
- 해당 람다만 종료시키려면 레이블을 사용한다.
    
    

## 타입 아규먼트를 reified로 사용할 수 있다.

제네릭 타입은 컴파일되면 제네릭과 관련된 내용이 제거된다. 

```kotlin
any is List<Int> //오류
any is List<*> //pass
```

```kotlin
inline fun <reified T> printTypeName() {
	print(T::class.simpleName)
}

printTypeName<Int>() //Int 
```

filterIsInstance도 특정 타입의 요소를 필터링할 때 사용한다.

## 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다.

함수 호출과 리턴을 위해 점프하고 백스택을 추적하는 과정이 없기 때문에 훨씬 빠르다. 하지만 함수 파라미터를 가지지 않는 함수에서는 큰 성능 차이가 없다. 

코틀린/JS 에서 함수를 일급 객체로 처리해서 간단히 변환이 이뤄지는 반면, 코틀린/JVM은 익명 클래스나 일반 클래스를 기반으로 함수를 객체로 만들어낸다.  그래서 함수 본문을 객체로 랩하면 코드의 속도가 느려지기 때문에 inline으로 본문을 바로 수행하면 빠른 것이다.

```kotlin
val lambda: () -> Unit = {~}

//-> 익명 클래스로 컴파일 
Function0<Unit> lambda = new Function0<Unit>() {
	public Unit invoke(){~}
}

//-> 일반 클래스로 컴파일 
public class Test$lambda implements Function0<Unit> {
	public Unit invoke() {~}
}
```

- () → Unit 은 Function0<Unit>으로 컴파일
- () → Int는 Function0<Int> 로 컴파일
- (Int) → Int 는 Finction1<Int, Int>로 컴파일
- (Int, Int) → Int는 Function2<Int, Int, Int>로 컴파일

특히 함수 리터럴 내부에서 지역 변수를 캡처하면 이 값은 래핑해야해서 큰 차이를 발생시킨다.

```kotlin
val l = 1L

noinlineRepeat(100_000_000) {
	l += it  //l을 직접 사용할 수 없다. 아래와 같이 래핑한 다음 사용한다.
}

val a = Ref.LongRef()
a.element = 1L
noinlineRepeat(100_000_000) {
	a.element = a.element + it
}
```

그래서 유틸리티 함수를 만들 때 컬렉션 처리같은 건 그냥 인라인을 붙여서 작성하는게 좋아서 대부분의 표준 라이브러리는 inline 으로 정의된다.

## 비지역적 리턴(non-local return)을 사용할 수 있다.

```kotlin
fun main() {
	repeatNoinline(10) {
		print(it)
		return //사용 불가 
	}
}
```

함수 리터럴이 컴파일될 때, 함수가 객체로 래핑되기 때문이다. 함수가 다른 클래스에 위치하기 때문에 return으로 메인에 돌아올 수 없다. Inline함수는 함수 내부에 박히기 때문에 이런 제약이 없다.

## inline 한정자의 비용

유용하긴 하지만 모든 곳에서 사용할 수 없다.

1. 재귀적으로 동작할 수 없다.
2. 가시성 한정자를 사용할 수 없다. 구현을 숨길 수 없기 때문이다.

```kotlin
inline fun printThree() {
	print(3) 
}

inline fun threPrintThree() {
	printThree()
	printThree()
	printThree()
}

//컴파일
inline fun threPrintThree() {
	print(3) 
	print(3) 
	print(3) 
}
```

너무 남용하면 코드의 크기가 쉽게 커진다. 

## crossinline과 noinline

- crossinline: 아규먼트로 인라인 함수를 받지만, non-local 리턴을 하는 함수는 받을 수 없게 만든다. 인라인으로 만들지 않은 다른 람다와 조합해서 사용할 때 활용한다.
- noinline: 아규먼트로 인라인함수를 받을 수 없게 만든다.

## 정리

인라인이 사용되는 사례

- print처럼 매우 많이 사용되는 함수인 경우
- filterIsInstance처럼 reified타입을 전달받는 경우
- 함수 타입 파라미터를 갖는 톱레벨 함수를 정의해야 하는 경우, 컬렉션 헬퍼함수(map, filter, flatmap, joinToString,..), 스코프함수(also, apply,..), 톱레벨 유틸리티 함수(repeat, run, with,..)

API를 정의할 땐 인라인 함수를 거의 사용하지 않는다.
