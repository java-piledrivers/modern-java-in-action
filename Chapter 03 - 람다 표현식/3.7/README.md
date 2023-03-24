# 3.7람다, 메서드 참조 활용하기

리스트를 다양한 정렬 기법으로 정렬하는 문제를, 더욱 간결하게 해결하는 방법을 정리한다.
2장부터 3장까지 다룬 동작 파라미터화, 익명 클래스, 람다 표현식, 메서드 참조를 통해 최종적으로 아래와 같은 코드가 만들어진다.

inventory.sort(comparing(Apple::getWeight));


해당 장의 예제에서는 List API의 sort 메서드를 사용한다.
sort 메서드는 아래와 같은 시그니처를 가지고 있다.

void sort(Comparator<? super E> c) ->오버로딩으로 여러개 존재하는 sort 메서드중 하나

Comparator 객체를 인수로 받아 두 사과를 비교하는데, Comparator를 어떻게 구현하는 방법에 따라 다양하게 정렬를 할 수 있다.

즉 2.2 에서 다룬 동작 파라미터화를 구현할 수 있다.


1. 동작 파라미터화(코드 전달) 예제 코드

```java


public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}

public class AppleComparatorReverse implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight()) * -1 ;
    }
}

...
//1단계 동작 파라미터화를 통해 런타임에서도 원하는 정렬 전략을 선택할 수 있다.[2.2]
inventory.sort(new AppleComparator());
//정렬 전략을 변경하여 역순으로 정렬한다.
inventory.sort(new AppleComparatorReverse());
```

만약 Comparator가 한번만 사용된다면, 코드를 구현하는 것보단 익명 클래스를 이용하여 별도의 클래스 선언 없이 코드를 생성할 수 있다.

```java

//2단계: 익명 클래스를 통해 별도의 클래스를 만들 필요없이 정렬 전략을 전달한다.
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
```


3장에서 다룬 것처럼 이러한 익명클래스 역시 더욱 경량화된 문법인 람다 표현식을 사용하여 대체할 수 있다.

이미 이번장에서 다룬 것처럼 
함수형 인터페이스(functional interface)는 하나의 추상 메서드(abstract method)만을 가지는 인터페이스다. 

람다 표현식(lambda expression)은 이 함수형 인터페이스의 구현을 쉽게 할 수 있게 해주는 문법이다. 

람다 표현식을 사용하면 특정 동작을 파라미터로 전달하는 코드를 위처럼 간결하게 작성할 수 있다.

Comparator 인터페이스는 하나의 추상 메서드 compare만을 가지는 함수형 인터페이스며, 반환값이 int이다.

int compare(T o1, T o2)

함수 디스크립터는 (T, T) -> int 이며, 현재 우리의 코드는 (Apple, Apple) -> int 로 표현할 수 있다.


```java
//3단계: 람다 표현식 사용
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

//형식 추론을 사용한 코드
//자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 활용해서 람다의 파라미터 형식을 추론한다.[3.5]
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

*함수 디스크립터는 함수형 인터페이스의 추상 메서드의 입력과 출력을 간략하게 표현하는 것이다.


이렇듯 람다 표현식을 사용하면, 코드를 간략하게 작성할 수 있음을 확인했다.
하지만 메서드 참조[3.6]를 사용하면 코드를 더욱 간략하게 줄일 수 있다.
 
```java
//4단계: 메서드 참조
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

```
바로 앞장에서 다룬 것처럼 메서드 참조를 사용하려면 
java.util.Comparator.comparing을 정적으로 임포트하고, comparing 메서드를 사용해야한다.

