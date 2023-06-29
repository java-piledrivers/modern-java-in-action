### 18.3 재귀와 반복

재귀는 함수형 프로그래밍의 한 기법이다.

순수 함수형 프로그래밍 언어에서는 while, for 같은 반복문을 포함하지 않는다. 반복문으로 인해 변화가 생길 수 있기 때문이다.

함수형 스타일에서는 지역 변수는 자유롭게 갱신할 수 있다.(변화를 알아차리지만 못한다면 아무 상관이 없다.)
```java
// 호출자는 변화를 알 수 없으므로 상관없다.
Iterator<Apple> it = apples.iterator(); // iterator()는 새로운 Itr 객체를 반환
while (it.hasNext()) {
    Apple apple = it.next(); 
    // ...
}
// 공유되는 stats의 상태가 변화되므로 문제가 발생할 수 있다.
public void searchForGold(List<String> list, Stats stats) {
    for (String string : list) {
        if ("gold".equals(string)) {
            stats.incrementFor("gold"); // stats가 다른 부분과 공유되고 있는 상태인데 반복문 안에서 상태가 변화되고 있음
        }
    }
}
```
이렇게 반복문을 사용할 경우, 함수형 프로그래밍이 깨질 수 있다. 이럴때 재귀를 사용하면 변화가 일어나지 않는다.(이론적으로 반복을 이용하는 모든 프로그램을 재귀로도 구현 가능)

```java
// 반복 방식 팩토리얼
public int factorialIterative(int n) {
    int r = 1;
    for (int i = 1; i <= n; i++) { 
        r *= i; 
    }
    return r;
}
// 재귀 방식 팩토리얼
public long factorialRecursive(long n) {
    if (n == 1) {
        return 1;
    }
    return n * factorialRecursive(n - 1); // 최종 연산이 n * 재귀 호출 결과값
}
// 스트림을 사용한 팩토리얼
public long factorialStreams(long n) {
    return LongStream.rangeClosed(1, n)
            .reduce(1, (a, b) -> a * b);
}
```
무조건 반복보다 재귀가 좋다고는 할 수 없다.
재귀코드가 자원을 더 많이 사용한다. 재귀는 호출될 때마다 호출 스택에 호출시 생성되는 정보를 저장할 스택 프레임이 만들어진다. 즉, 입력값에 따라 만들어지는 스택 프레임이 늘어나므로 메모리 사용량이 증가한다.

이 문제를 해결하기 위해 꼬리 호출 최적화라는 해결책을 제공해준다.

```java
// 꼬리 재귀 팩토리얼
public long factorialTailRecursive(long n) {
    return factorialHelper(1, n);
}

private long factorialHelper(long acc, long n) {
    if (n == 1) {
        return acc;
    }
    return factorialHelper(acc * n, n - 1); // 최종 연산이 재귀호출
}
```
일반 재귀는 중간 결과를 각각의 스택 프레임으로 저장해야하지만, 꼬리 재귀는 컴파일러가 하나의 스택 프레임을 재활용할 수 있다.(재귀 호출이 최종 연산일 경우, 스택에 있는 결과값을 교체하는 식)

자바는 이런 최적화를 제공하지 않지만 꼬리 재귀를 사용해야 추가적인 컴파일러 최적화를 기대할 수 있다.

**자바에서 꼬리 재귀 최적화를 지원하지 않는 이유**
>jdk 클래스들에는 몇몇 보안에 민감한 메소드들이 있는데, 이 메소드들은 메소드 호출을 누가 했는지를 알아내기 위해 jdk 라이브러리 코드와 호출 코드간의 스택 프레임 갯수에 의존한다. 스택 프레임의 수의 변경을 유발하게 되면 이 의존관계를 망가뜨리게 되고 에러가 발생할 수 있다. 이게 멍청한 이유라는 것을 인정하며, JDK 개발자들은 이 메커니즘을 교체해 오고 있다.
그리고 추가적으로, tail recursion이 최상위 우선순위는 아니지만,
결국에는 지원될 것이다.
