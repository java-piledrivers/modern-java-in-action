# 11.2 Optional 클래스

자바 8에서 도입된 `java.util.Optional<T>` 클래스는 하스켈이나 스칼라와 같은 함수형 프로그래밍 언어에서 영감을 얻어서 만들어졌다. 하지만, 이 클래스를 올바르게 이해하고 사용하기 위해서는 그 배경에 대한 조금 더 깊은 이해가 필요하다. 

## 함수형 프로그래밍 언어에서의 Null

하스켈이나 스칼라 같은 함수형 프로그래밍 언어에서는 `null`이라는 개념이 존재하지 않는다. 대신에 이들 언어는 값이 존재하지 않는 상황을 표현하기 위해 "Option"이라는 특수한 자료형을 사용한다. 예를 들어, 스칼라에서는 `Option[T]`라는 타입을 사용해서 값이 존재할 수도, 존재하지 않을 수도 있는 상황을 표현한다. `Option[T]`은 `Some[T]` 또는 `None` 두 가지 값만을 가질 수 있다.

## 자바에서의 Optional

이와 같은 방식을 자바에서 도입한 것이 바로 `Optional<T>`이다. `Optional<T>`은 `T` 타입의 값을 가질 수도, 가질 수 없을 수도 있는 컨테이너이다. 이를 사용하면 메서드가 값이 없는 상황을 명시적으로 표현할 수 있으며, 이를 통해 NullPointerException을 방지할 수 있다.

그러나 모든 `null` 참조를 `Optional`로 대치하는 것은 바람직하지 않다. `Optional`의 주요 역할은 더 이해하기 쉬운 API를 설계하는 것이다. 즉, 메서드의 시그니처만 보고도 반환값이 선택형 값인지 여부를 쉽게 알아볼 수 있다. `Optional`이 반환되는 메서드는 값이 없을 수 있는 상황에 대응하도록 강제한다.

## Optional의 사용법

```java
public class Person {
	private Optional<Car> car; // 사람이 차를 소유했을 수도 소유하지 않았을 수도 있으므로 Optional 정의
	public Optional<Car> getCar() {
		return car;
	}
}

public class Car {
	private Optional<Insurance> insurance; // 자동차가 보험에 가입되어 있을 수도, 가입되어 있지 않았을 수도 있으므로 Optional 정의 
	public Optional<Insurance> getInsurance() {
		return insurance;
	}
}

public class Insurance {
	private String name; // 보험회사는 밥ㄴ드시 이름이 있다
	public String getName() {
		return name;
	}
}
```

```java
Optional<Car> optionalCar = person.getCar();
if (optionalCar.isPresent()) {
	Car car = optionalCar.get();
	// 이제 car를 사용할 수 있다.
} else {
	// car가 없는 경우에 대한 처리
}
```
이런 식으로 `Optional`에 저장된 값을 사용할 수 있다. `isPresent()` 메서드는 `Optional`이 값을 가지고 있는지 여부를 확인하고, `get()` 메서드는 그 값을 반환한다. 만약 `Optional`이 값을 가지고 있지 않은 상태에서 `get()`을 호출하면 NoSuchElementException이 발생한다.

그래서 바로 `get()`을 호출하는 것은 바람직하지 않다. 위 예시처럼 if-else 에 `isPresent()`를 넣는 것은 마치 `if (car != null)`와 다를 바가 없다. `Optional`을 올바르게 사용하는 방법은 [이 블로그](https://mangkyu.tistory.com/203)에 잘 정리되어 있다.

링크에 정리되어 있는 내용을 정리하면 아래와 같으니 흥미로운 부분은 읽어볼만한 것 같다.

```
- Optional 변수에 Null을 할당하지 말아라
- 값이 없을 때 Optional.orElseX()로 기본 값을 반환하라
- 단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라
- 생성자, 수정자, 메소드 파라미터 등으로 Optional을 넘기지 마라
- Collection의 경우 Optional이 아닌 빈 Collection을 사용하라
- 반환 타입으로만 사용하라
```

## Optional의 람다 표현식과 함께 사용

```java
person.getCar().ifPresent(car -> {
	// car를 사용하는 코드
});
```
`ifPresent()` 메서드는 `Optional`에 값이 존재하는 경우, 주어진 람다 표현식을 실행한다. 이 메서드를 이용하면 값이 존재하지 않는 경우를 특별히 처리하지 않아도 된다.

## Optional 메서드 체이닝

`Optional`은 메서드 체이닝을 지원하여 편리하게 사용할 수 있다. `map`과 `flatMap` 메서드는 `Optional`에 저장된 값을 변환하거나, 다른 `Optional`로 대체하는 데 사용된다.

```java
Optional<Insurance> optInsurance =
        person.getCar()
              .flatMap(Car::getInsurance);

optInsurance.ifPresent(insurance -> System.out.println(insurance.getName()));
```

위 예시에서, `flatMap` 메서드는 `Car` 객체에 있는 `getInsurance` 메서드를 호출하고, 그 결과로 나온 `Optional<Insurance>`을 반환한다. `ifPresent` 메서드를 통해 값이 존재하는 경우에만 이름을 출력하도록 한다.

## Optional과 Null 안전성

`Optional`은 Null 안전성을 제공한다. `Optional`을 사용하면 프로그램에서 Null을 참조하는 실수를 방지할 수 있다. 예를 들어, 메서드가 `null`을 반환하는 경우에 클라이언트 코드에서는 `null` 체크를 항상 해야 하지만, `Optional`을 사용하면 `null` 체크 없이 안전하게 코드를 작성할 수 있다. 

## Optional을 사용하면 좋은 상황

1. **값이 없을 수 있는 상황을 명확히 표현하고 싶은 경우** 메서드 시그니처만 보고도 값의 존재 여부를 판단할 수 있게 하려면 `Optional`을 사용해야 한다.

2. **NullPointerException을 방지하고 싶은 경우** 메서드가 `Optional`을 반환하면 클라이언트 코드는 반환값이 `null`인지 아닌지 확인할 필요가 없다. 따라서 NullPointerException을 방지할 수 있다.

3. **메서드 체인을 사용하고 싶은 경우** `Optional`의 메서드 체이닝은 코드를 깔끔하게 유지하는 데 도움이 된다.

## Optional의 단점

1. **Optional 객체는 Null에 비해 더 많은 메모리를 사용한다.** `Optional` 객체는 내부적으로 참조를 하나 더 갖고 있기 때문에 `null`에 비해 메모리 사용량이 약간 더 많다. 따라서, 메모리가 아주 제한적인 상황에서는 `Optional`의 사용을 잘 고려해야 한다.

2. **Optional 사용은 코드 복잡성을 증가시킬 수 있다.** `Optional`의 잘못된 사용은 오히려 코드를 복잡하게 만들 수 있다. 예를 들어, `Optional`을 사용해야 하는 적절한 상황을 판단하지 못하거나, `Optional`의 메서드 체이닝을 남용하면 코드가 난해해질 수 있다. 

```java
public String getCarInsuranceName(Person person) {
    if (person.getCar().isPresent()) {
        if (person.getCar().get().getInsurance().isPresent()) {
            return person.getCar().get().getInsurance().get().getName();
        }
    }
    return "Unknown";
}

```

3. **필드로서의 Optional 사용에는 주의가 필요하다.** 일반적으로, Optional은 메서드의 반환 타입으로 사용하는 것이 적절하며, 필드로서 Optional을 사용하는 것은 권장되지 않는다. 필드로 Optional을 사용하는 것은 메모리 부하와 직렬화 문제를 초래할 수 있다. 또, JPA와 같은 ORM 과 사용할 수 없다. Optional이 직렬화를 지원하지 않기 때문이다.

```java
@Entity
public class Person {
    @Id
    @GeneratedValue
    private Long id;
    
    private Optional<Car> car; 

    public Optional<Car> getCar() {
        return car;
    }
}
```


## Optional과 Stream API

자바 8의 Stream API와 Optional은 잘 어울린다. Stream API에서는 원소의 시퀀스를 처리하는데, 이 시퀀스의 원소가 없는 경우에도 Optional을 반환함으로써 NullPointException을 방지한다.

```java
List<String> names = Arrays.asList("John", "Jane", "Tom", "Emily");
Optional<String> firstPersonWithThreeLettersName = 
    names.stream()
         .filter(name -> name.length() == 3)
         .findFirst();
```

위의 예제에서, `filter` 메서드는 이름의 길이가 3인 사람들의 스트림을 생성하고, `findFirst` 메서드는 이 중 첫 번째 원소를 찾아서 `Optional`로 반환한다. 만약 조건에 맞는 원소가 없는 경우에도 `Optional.empty`가 반환되므로, NullPointerException을 방지할 수 있다.

결론적으로, Optional은 코드를 더 안전하고 이해하기 쉽게 만드는 데 매우 유용한 도구이다. 하지만 그 사용에는 주의가 필요하며, 적절한 상황에서만 사용해야 한다.
