# 13.3 디폴트 메서드 활용 패턴

디폴트 메서드를 이용하는 두 가지 방식이 있다.

- 선택형 메서드 optional method
- 다중 상속 multiple inheritance of behavior

## 13.3.1 선택형 메서드

Iterator 인터페이스는 remove 메서드를 제공하는데,
자바 8 이전에는 필요가 없어도 구현을 해야했다.

자바 8 부터 기본 구현이 제공되면서
사람들이 사용을 잘 하지않는 remove 메서드를 구현할 필요가 없어졌다.

따라서, 선택적으로 remove 를 구현 할 수 있게 되어졌다.

## 13.3.2 동작 다중 상속

자바는 기본적으로 하나의 클래스만 상속을 받을 수 있지만,   
인터페이스는 여러개 구현할 수 있기 때문에 아래와 같은 게 가능하다.

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

ArrayList 는 한 개의 클래스를 상속 받고 4개의 인터페이스를 구현하지만,
결과적으로 다중 상속이 가능하게 되어진다.

![2023-05-27_08-02-22](https://github.com/java-piledrivers/modern-java-in-action/assets/59721293/4a7e7b27-5a46-4af0-b1e7-f6a6d4fba46d)

예를 들어, ArrayList 가 Collection 을 구현하고 있진 않지만,
AbstractList 상속받고, 이어서 AbstractCollection 을 상속받고 있기 때문에,
AbstractionCollection 에 의해서 Collection 을 상속받을 수 있는 것이다.


## 옳지 못한 상속

한 개의 메서드를 재사용하려고 100개의 메서드와 필드가 정의되어 있는 클래스를 상속받는 것은 좋은 생각이 아니다
이럴땐 상속대신 delegation 즉, 멤버 변수를 이용해서 클래스에서 필요한 메서드를 직접 호출하는 메서드를 작성하는 것이 좋다

예를 들어, 100개의 메서드와 필드가 정의되어 있는 클래스를 상속받아야 할 때, delegation을 사용하여 필요한 기능을 직접 호출하는 클래스를 작성할 수 있다.
이렇게 함으로써 필요한 메서드만을 선택적으로 호출하고 필요한 데이터만을 사용할 수 있다.

### delegation 예시

```java
public class CalculatorDelegate {
    private Calculator calculator; // Calculator 클래스의 인스턴스를 멤버 변수로 가짐

    public CalculatorDelegate() {
        calculator = new Calculator(); // Calculator 인스턴스를 생성하여 멤버 변수에 할당
    }

    public int add(int a, int b) {
        return calculator.add(a, b); // Calculator 클래스의 add 메서드 호출
    }

    public int subtract(int a, int b) {
        return calculator.subtract(a, b); // Calculator 클래스의 subtract 메서드 호출
    }

    // 필요한 다른 메서드들을 구현
}
```

