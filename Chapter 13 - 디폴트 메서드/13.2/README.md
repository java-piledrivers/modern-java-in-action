# 13.2 디폴트 메서드란 무엇인가?
- 자바 8에서는 호환성을 유지하면서 API를 바꿀 수 있도록 새로운 기능인 디폴트 메서드를 제공한다.
- 인터페이스를 구현하는 클래스에서 메서드를 구현하지 않아도 되는 새로운 메서드 시그너처이다.
- 인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 인터페이스 자체에서 기본으로 제공한다.
```java
public interface Sized {
    int size();
    default boolean isEmpty() {
        return size() == 0;
    }
}
위 코드에서 Sized 인터페이스는 추상 메서드 size와, 디폴트 메서드 isEmpty를 포함한다.
Sized 인터페이스를 구현하는 모든 클래스는 isEmpty의 구현까지 상속받는다.
```

- Collection 인터페이스의 stream, List 인터페이스의 sort 역시 디폴트 메서드다.
- Predicate, Function, Comparator 등 많은 함수형 인터페이스도 <br>Predicate.and, Function.andThen 등 다양한 디폴트 메서드를 포함한다.<br>
  (함수형 인터페이스는 오직 하나의 추상 메서드를 포함한다. -> 디폴트 메서드는 추상 메서드에 해당하지 않는다.)
<br>

#### 추상 클래스와 인터페이스 비교
- 둘 다 추상 메서드와 바디를 포함하는 메서드를 정의 할 수 있다.
- 추상 클래스는 하나만 상속 받을 수 있지만, 인터페이스는 여러 개 구현할 수 있다.
- 추상 클래스는 인스턴스 변수를 갖지만, 인터페이스는 인스턴스 변수를 가질 수 없다.
<br>

## 퀴즈 13-1
주어진 프레디케이트와 일치하는 모든 요소를 제거하는 removeIf 메서드를, 기존 컬렉션 API에 가장 적절히 추가하는 방법은?
<br>  -> 컬렉션 인터페이스에 아래의 디폴트 메서드를 추가한다.
```java
default boolean removeIf(Predicate<? super E> filter) {
    boolean removed = false;
    Iterator<E> each = iterator();
    while(each.hasNext()) {
        if(filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```
