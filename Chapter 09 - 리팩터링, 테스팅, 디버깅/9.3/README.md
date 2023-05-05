# 람다 테스팅

## 보이는 람다 표현식의 동작 테스팅

람다는 익명 함수이므로 테스트 코드 이름을 호출할 수가 없기 때문에 필드에 저장해서 재사용하거나 람다의 로직을 테스트할 수 있다.

아래는 Point 클래스에 compareByXAndThenY라는 정적 필드를 추가하고 생성된 인스턴스의 동작으로 람다 표현식을 테스트하는 예제이다.

```java
public class Point {
    public final static Comparator<Point> compareXAndThenY = comparing(Point::getX).thenComparing(Point::getY);
}
```

```java
@Test
public void testComparingTwoPoints() throws Exception {
    Point p1 = new Point(10, 15);
    Point p2 = new Point(10, 20);
    int result = Point.compareByXAndThenY.compare(p1, p2);
    assertTrue(result < 0);
}
```

## 람다를 사용하는 메서드의 동작에 집중하라

정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각을 캡슐화하는 람다의 목적에 따라, 람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.

## 복잡한 람다를 개별 메서드로 분할하기

많은 로직을 포함하는 복잡한 람다 표현식을 테스트해아 한다면, 람다 표현식을 메서드 참조로 바꾸어 새로운 일반 메서드로 선언하여 일반 메서드를 테스트하듯이 람다 표현식을 테스트할 수 있다.

## 고차원 함수 테스팅

고차원 함수, 즉 함수를 인수로 받거나 다른 함수로 반환하는 메서드는 비교적 사용하기 어려운데 메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트할 수 있다.

```java
@Test
public void testFilter() throws Exception {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    List<Integer> even = filter(numbers, i -> i % 2 == 0);
    List<Integer> smallerThanThree = filter(numbers, i -> i < 3);
 
    //다양한 Predicate로 filter 메서드 테스트
    assertEquals(Arrays.asList(2, 4), even);
    assertEquals(Arrays.asList(1, 2), smallerThanThree);
}
```
