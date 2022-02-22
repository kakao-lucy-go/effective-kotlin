서버가 원하는 결과를 만들어내지 못하는 경우가 있다.

1. 서버로부터 데이터를 읽어들이지 못했을 경우
2. 조건에 맞는 첫 번째 요소를 찾으려고 했는데 조건에 맞는 요소가 없는 경우 
3. 텍스트 파싱 형식이 안맞는 경우

이런 경우에 보통 아래와 같이 처리한다.

1. null 이나 실패를 나타내는 sealed 클래스(보통 Failure 라는 이름을 사용)를 리턴
2. 예외 throw 

예외는 잘못된 특별한 상황을 나타내야 하며 예외적인 상황이 발생했을 때 써야한다. 코틀린의 모든 예외는 unchecked 예외여서 문서에도 제대로 드러나지 않는다. 그래서 try catch를 쓰면 컴파일러가 할 수 있는 최적화가 제한된다.

```kotlin
checked: 사용자가 반드시 처리하도록 강제하는 예외
unchecked: 처리하지 않아도 실행에 문제가 없는 예외
```

null과 Failure가 오류를 표현할 때 가장 효율적이고 간단하고 명시적이다. 따라서 충분히 예측 가능한 범위의 오류는 null 과 Failure를 사용하고 예측하기 어려운 예외는 throw로 처리하는 것이 좋다.

```kotlin
inline fun <reifed T> String.readObjectoOrNull(): T? {
	if(incorrectSign) return null
	return result
}

inline fun <reified T> String.readObject(): Result<T> {
	if(incorrectSign) return Failure(JsonParsingException())
	return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()
```

위와 같이 작성하면 좋다.

```kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
//엘비스 연산자로 널 안정성 기능 사용 

val age = when(person) {
	is Success -> person.age
	is Failure -> -1
}
//when 표현식으로, 리턴된 Result와 같은 공용체(union type)를 처리
```

try-cach 보다 효율적이고 예외를 다루기 쉽다.  추가적인 정보를 처리해야한다면 sealed result를 사용하고 그렇지 않으면 null 을 처리하는 것이 일반적이다. 

일반적으로 두 가지 형태의 함수를 사용한다. 하나는 예상할 수 있을 때, 다른 하나는 예상할 수 없을때 사용한다.

1. get : 특정 위치에 있는 요소를 추출. 없으면 IndexOutOfBoundsException 발생 
2. getOrNull: out of range 오류가 발생하면 null 리턴

비슷하게 getOrDefault도 있다. 보통 getORNull + Elvis 연산자를 사용한다.

null 이 될 수 있음을 함수명에 작성하는 것이 좋다.
