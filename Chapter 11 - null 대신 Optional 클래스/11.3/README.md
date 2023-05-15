# Optional 적용 패턴

## Optional 객체 만들기

- 빈 Optional
  
  - Optional.empty로 빈 Optional 객체를 얻을 수 있다.
    ```java
    Optional<Car> optCar = Optional.empty();
    ```
    

- null이 아닌 값으로 Optional 만들기
  
  - Optional.of로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.
    ```java
    Optional<Car> optCar = Optional.of(car);
    ```
- null값으로 Optional 만들기
  
  - Optional.ofNullable로 null값을 저장할 수 있는 Optional을 만들 수 있다.
    ```java
    Optional<Car> optCar = Optional.ofNullable(car);
    ```

## Map으로 Optional의 값을 추출하고 변환하기

객체의 속성 값에 접근하기 전에 해당 객체가 null인지 확인해야할 필요가 있는데, 이러한 유형의 패턴에 사용할 수 있도록 Optional은 map 메서드를 지원한다.

아래 코드는 보험회사의 이름을 추출하는 코드인데, 객체가 비어있는 경우에는 아무 일도 일어나지 않고 비어있지 않으면 map의 인수로 제공된 함수가 값을 바꾼다.

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

## flatMap으로 Optional 객체 연결

다음과 같이 작성한 코드는 컴파일되지 않는다.

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson.map(Person::getCar).map(Car::getInsurance).map(Insurance::getName);
```

optPeople의 형식은 Optional<People>이므로 map 메서드를 호출할 수 있지만, map의 연산 결과는 중첩 Optional 형태이므로 getInsurance 메서드를 지원하지 않는다.

이 문제를 해결하기 위해서 flatMap을 이용할 수 있다. 보통 인수로 받은 함수를 스트림의 각 요소에 적용하면 스트림의 스트림이 만들어지는데, flatMap은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 남긴다.

아래 코드는 Optional로 자동차의 보험회사 이름을 짓는 코드이다.

```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
                 .flatMap(Car::getInsurance)
                 .map(Insurance::getName)
                 .orElse("Unknown");
}
```

## Optional 스트림 조작

자바 9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다.

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
    return persons.stream()
                  .map(Person::getCar) //사람 목록을 각 사람이 보유한 자동차의 Optional<Car> 스트림으로 변환
                  .map(optCar -> optCar.flatMap(Car::getInsurance)) //flatmap 연산을 이용해 Optional<Insurance>로 변환
                  .map(optIns -> optIns.map(Insurance::getName)) //해당 이름의 Optional<String>으로 매핑
                  .flatMap(Optional::stream) //Stream<Optional<String>>을 현재 이름을 포함하는 Stream<String>으로 변환
                  .collect(toSet()); //결과 문자열을 중복되지 않은 집합으로 수
}
```

## 디폴트 액션과 Optional 언랩

Optional 클래스는 인스턴스에 포함된 값을 읽는 다양한 방법을 제공하고 있다.

- get
  
  - 값을 읽는 가장 간단한 메서드이면서 동시에 가장 안전하지 않은 메서드이다.
    래핑된 값이 있으면 해당 값을 반환하고, 값이 없으면 NoSuchElementException을 발생시킨다.
- orElse
  
  - Optional이 값을 포함하지 않을 때 기본 값을 제공할 수 있다.
- orElseGet
  
  - orElse에 대응하는 게으른 버전의 메서드로, Optional에 값이 없을 때만 인수로 받는 Supplier가 실행된다.
- orElseThrow
  
  - Optional이 비어있을 때 예외를 발생시킨다는 점에서 get과 비슷하지만, 해당 메서드는 발생시킬 예외의 종류를 선택할 수 있다.
- ifPresent
  
  - 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있고, 없으면 아무 일도 일어나지 않는다.
- ifPresentOrElse
  
  - Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다는 점 이외에는 ifPresent와 동일한 기능을 한다.

## 두 Optional 합치기

Person과 Car 정보를 이용해 가장 저렴한 보험료를 제공하는 보험회사를 찾는 비즈니스 로직을 구현한 외부 서비스가 있다고 가정하자.

```java
public Insurance findCheapestInsurance(Person person, Car car) {
    //다양한 보험회사가 제공하는 서비스 조회
    //모든 결과 데이터 비교
    return cheapestCompany;
```

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

첫 번째 Optional이 비어있다면 그대로 빈 Optional을 반환하고, 비어있지 않다면 Optional<Insurance>를 반환하는 메서드의 입력으로 person을 사용한다.

결과적으로 person과 car가 모두 존재하면 map 메서드로 전달한 람다 표현식이 findCheapestInsurance 메서드를 안전하게 호출할 수 있다.

## 필터로 특정 값 거르기

예를 들어 보험회사 이름이 'CambridgeInsurance'인지 확인해야한다고 가정할 때, 작업을 안전하게 수행하려면 객체가 null인지 확인한 다음 메서드를 호출해야 한다.

이때 Optional 객체에 filter 메서드를 이용해서 다음과 같이 코드를 작성할 수 있다.

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"));
```

filter 메서드는 프레디케이트를 인수로 받는데, Optional 객체가 값을 가지면서 프레디케이트와 일치하면 filter 메서드는 그 값을 반환하고 그렇지 않으면 빈 Optional 객체를 반환한다.
