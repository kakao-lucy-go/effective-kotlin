가능하다면 직접 오류를 정의하는 것보다 최대한 표준 라이브러리의 오류를 사용하는 것이 좋다. 왜냐하면 많은 개발자가 알고있기 때문이다.

1. IllegalArgumentException, IllegalStateException: require, check를 사용
2. IndexOutOfBoundException: 인덱스 파라미터 범위 초과 
3. ConcurrentModificationException: 동시 수정을 금지했는데 발생
4. UnsupportedOperionException: 사용자가 사용하려고 했단 메소드가 현재 객체에서는사용할 수 없다. 기본적으로는 사용할 수 없는 메소드는 클래스에 없는 것이 좋다. 인터페이스 분리 원칙은 클라이언트가 자신이 사용하지 않는 메소드에 의존하면 안된다는 원칙이고 이 원칙을 위반하는 경우가 된다.
5. NoSuchElementException: 사용자가 사용하려고 했던 요소가 없다.
