‘개발자는 코드를 작성하는 데는 1분 걸리지만, 이를 읽는 데는 10분이 걸린다' - 로버트 마틴 <클린 코드>

## 11.1 인식 부하 감소

```kotlin
//구현 A
if(person != null && person.isAdult) {
	view.showPerson(person)
} else {
	view.showError()
}

//구현 B
person?.takeIf { it.isAdult }?.let(view::showPerson) ?: view.showError()
```

B가 더 짧긴 하지만 A의 가독성이 더 좋고 수정이 용이하고 디버깅이 간단하다. ‘인지 부하'를 줄이는 방향으로 코드를 작성하는 것이 좋다.

## 11.2 극단적이 되지 않기

let을 사용하는 경우는

1. 연산을 마지막에 할 때 (아규먼트 처리 후)
2. 데코레이터를 사용해서 객체를 래핑할 때 (InputStream을 concrete한 객체들로 래핑한다)

이다.

```kotlin
students
.filter { it.result >= 50 }
.let(::print)

var obj = FileInputStream("/file.gz")
            .let(::BufferedInputStream)
            .let(::ZipInputStream)
            .let(::ObjectInputStream)
            .readObject() as SomeObject
```

이 코드들은 디버깅이 어렵고 가독성도 좋지 않지만 비용을 지불할 가치가 있기 때문에, 정당한 이유가 있다면 사용해도 된다.

## 11.3 컨벤션

컨벤션에 맞게 개발해야 가독성이 좋아진다.

1. 연산자를 의미에 맞게 사용한다
2. 이미 있는 기능을 또 만들 필요는 없다.
3. invoke 연산자와 ‘람다를 마지막 아규먼트로 사용한다'라는 컨벤션을 함께 쓰는 것은 신중해야 한다.
