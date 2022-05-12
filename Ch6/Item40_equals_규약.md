# 아이템 40. equals의 규약을 지켜라

Any는 다음의 메소드를 갖는다.

- equals (==)
- hashCode
- toString

‘아이템 32: 추상화 규약을 지켜라' 에서 처럼 Any 를 상속받는 모든 메소드는 이 규약을 잘 지키는 것이 좋다.

## 동등성

- 구조적 동등성: ==로 확인하는 동등성.
- 레퍼런스적 동등성: ===로 확인하는 동등성. 같은 객체를 가리키면 true

모든 객체에서 사용 가능하지만 다른 타입의 두 객체를 비교하는 것은 허용하지 않는다.

다만, 상속 관계를 갖는다면 가능하다.

## equals가 필요한 이유

Any는 ===로 완전히 같은 객체인지 확인한다. 이는 모든 객체는 디폴트로 유일한 객체라는 것을 의미한다.

예를 들어 기본 생성자의 프로퍼티가 같으면 같은 객체로 보는 형태라면 data를 붙여서 정의하면 자동으로 그 기능으로 동작한다.

일반적으로 코틀린에서 equals를 직접 구현할 필요가 없지만 일부 프로퍼티만 같은지 확인해야 하는 경우에는 직접 구현을 한다.

- 기본적으로 제공되는 동작과 다른 동작을 해야 하는 경우
- 일부 프로퍼티만으로 비교해야하는 경우
- data한정자가 붙는 걸 원하지 않거나, 비교해야 하는 프로퍼티가 기본 생성자에 없는 경우

## equals의 규약

- 반사적 동작. null 이 아닌 값은 x.equals(x)는 true이다.
- 대칭적 동작: x와 y가 null 이 아니라면 x.equals(y) 와 y.equals(x)의 결과물은 같다.
- 연속적 동작: x,y,z가 null 이 아니고 x.equals(y) 와 y.equals(z)가 true라면 x.equals(z)도 true다.
- 일관적 동작: x와 y가 null 이 아니라면 x.equals(y)를 여러번 수행해도 항상 같은 결과이다.
- 널과 관련된 동작: x가 null 이 아니라면 x.equals(null)은 항상 false이다.

추가로, equals, toString, hashCode는 빠르게 동작해야 한다.

- equals는 **반사적** 동작을 해야한다. 예를 들어 x.equals(x)가 true이다.

```kotlin
class Time(
	val milliArg: Long = -1,
	val isNow: Boolean = false
) {
	val millis: Long get() = if(isNow) System.currentTimeMillis() else millisArg
	override fun equals(other: Any?): Boolean = other is Time && millis == other.millis
}

//이면 equals를 했을 때 true일 수도 있고 false일 수도 있다.
```

이는 고전적으로는 태그 클래스로 해결할 수 있는데 아이템 39처럼 sealed 클래스로 해결할 수 있다.

```kotlin
sealed class Time
data class TimePoint(val millis: Long): Time()
object Now: Time()
```

- equals는 대칭적 동작을 해야한다.

```kotlin
Complex(1.0, 0.0).equals(1.0) //true
1.0.equals(Complex(1.0, 0.0)) //false
```

- 객체 동등성이 연속적이어야 한다.

연속적이지 않은 경우에 처음부터 상속 대신 컴포지션을 사용해서 두 객체를 아예 비교하지 못하게 하는 것이 좋다.( 아이템 36: 상속보다 컴포지션을 사용하라 - 객체를 프로퍼티로 갖는 형태)

- equals는 반드시 일관성을 가져야 한다. 반드시 비교 대상이 되는 두 객체에만 의존하는 순수 함수여야 한다.
- null 과 같을 수 없다. x가 null 이 아닐 때, 모든 x.equals(null)은 false이다. null은 유일한 객체이기 때문이다.

## URL 과 관련된 equals 문제

URL은 동일한 IP 로 해석이 되면 true이다. 즉, 인터넷 연결이 끊겨 있으면 false가 될 수 있다.

- 일관성이 깨짐.
- 네트워크 처리를 하기 때문에 느림
- 동일한 IP 를 갖는다고 해서 동일한 콘텐츠는 아니다. 가상 호스팅을 한다면 관련 없는 사이트가 같은 IP를 공유할 수도 있다.

안드로이드는 추후 수정되었다.

## equals 구현하기

웬만하면 직접 구현하는 것은 좋지 않다. 구현해야 한다면 이 클래스는 final 로 만드는 것이 좋고, 상속을 해야한다면 서브클래스에서 equals 구현 방식을 변경하면 안된다.

참고로 data 클래스는 언제나 final 이다.
