코틀린의 널 안정성은 주요 기능인데 만약 자바에서 String 타입을 리턴하는 메소드가 있다면 코틀린은 어떻게 사용해야 할까?

@Nullable 이 붙어있다면 String? 으로 변경하고 @NotNull 이라면 String으로 변경한다. 하지만 아무것도 붙어있지 않다면 자바는 모든 것이 nullable일 수 있다.

null 이 아님이 확실하다면 !!을 붙여서 단정지을 수 있다. 하지만 List<User>, List<List<User>>, List<List<List<User>>> 처럼 내부 데이터도 null 일 수 있음을 거듭 확인해야하는 복잡한 프로퍼티인 경우에는 플랫폼 타입(!)을 사용할 수 있다. 직접적으로 코드에 나타나진 않지만 아래와 같이 선택적으로 사용한다.

```kotlin
//자바
public class UserRepo {
	public User getUser()~
}

//코틀린
val repo = UserRepo()
val user1 = repo.user //User! 타입
val user2: User = repo.user //User 타입
val user3: User? = repo.user //User? 타입 
```

하지만 null 이 아니라고 생각했는데 null 일 수 있으므로 안전하진 않다.

- JetBrains, Android 등에서 지원하는 @Nullable, @NotNull 기능의 어노테이션들이 있다. 우리는 젯브레인만 알 면 될 것 같아서 굳이 다 작성하진 않겠다. P30 참조

```kotlin
//자바
public class JavaClass {
	public String getValue() { return null; }
}

//코틀린
fun statedType() {
	val value: String = JavaClass().value //자바에서 값을 가져오는 위치에서 NPE
	println(value.length)
}

fun platformType() {
	val value = JavaClass().value
	println(value.length) //값을 활용할 때 NPE
}
```

플랫폼 타입으로 지정된 변수는 nullable일 수도 있고 아닐수도 있어서 몇 번 안전하게 사용했다 하더라도 NPE가 발생할 수 있어서 추적이 어렵다.

## 정리

nullable 여부를 알 수 없는 타입을 플랫폼 타입이라고 하고 이를 사용하면 안전하지 않기 떄문에 제거하고, 자바 코드에는 nullable 여부를 지정하는 어노테이션을 활용하면 좋다.
