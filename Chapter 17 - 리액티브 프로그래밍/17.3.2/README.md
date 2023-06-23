# 17.3.2 Observable을 변환하고 합치기

RxJava는 자바를 위한 리액티브 프로그래밍 라이브러리로 데이터 스트림과 변화를 관찰할 수 있도록 하는 비동기 프로그래밍 패러다임을 제공한다.

## 기본 개념

- **리액티브 프로그래밍**: 데이터 흐름과 변화 전파에 중점을 둔 프로그래밍 패러다임이다.
- **Observable**: 데이터의 시퀀스를 나타내며 이 시퀀스에 대해 연산을 수행하고 변화를 감시하는데 사용된다.

## Observable과 연산자들

### map

- `map` 연산자를 사용하여 Observable에서 방출되는 각 항목에 대해 특정 함수를 적용한다.
- 예시: 화씨 온도를 섭씨로 변환하는 코드.

```java
public static Observable<TempInfo> getCelsiusTemperature(String town) {
    return getTemperature(town)
        .map(temp -> new TempInfo(temp.getTown(), (temp.getTemp() - 32) * 5 / 9));
}
```

이 코드는 `getTemperature` 메서드를 통해 받은 Observable에서 방출되는 각 온도를 섭씨로 변환하여 새로운 Observable로 반환한다.

### filter

- `filter` 연산자를 사용하여 Observable에서 방출되는 항목들 중 조건에 맞는 것들만 선택한다.
- 예시: 섭씨 0도 이하인 온도만 선택하는 코드.

```java
public static Observable<TempInfo> getNegativeTemperature(String town) {
    return getCelsiusTemperature(town)
        .filter(temp -> temp.getTemp() < 0);
}
```

이 코드는 앞서 변환한 섭씨 온도 중에서 0도 이하인 온도만 필터링하여 새로운 Observable로 반환한다.

### merge

- `merge` 연산자를 사용하여 여러 Observable을 하나로 합친다.
- 이 연산자는 여러 Observable에서 방출되는 항목들을 단일 Observable로 합친다.
- 예시: 여러 도시의 온도를 하나의 Observable로 합치는 코드.

```java
public static Observable<TempInfo> getCelsiusTemperatures(String... towns) {
    return Observable.merge(Arrays.stream(towns)
        .map(TempObservable::getCelsiusTemperature)
        .collect(Collectors.toList()));
}
```

이 코드는 입력으로 받은 도시 목록에 대해 각 도시의 섭씨 온도를 Observable로 가져와서 하나의 Observable로 합친다.

## 장점

- RxJava는 자바 9의 Flow API에 비해 스트림을 생성하고, 변형하고, 조합하는 데 풍부한 도구를 제공한다.
- 한 스트림을 다른 스트림의 입력으로 사용하여 변환, 필터링, 병합 등 다양한 방식으로 스트림을 조작할 수 있다.
- 리액티브 스트림 커뮤니티는 복잡한 합치기 함수들을 시각적으로 표현하기 위해 마블 다이어그램이라는 시각적 방법을 사용한다. 이를 통해 함수의 작동 방식을 더 쉽게 이해할 수 있다.

## 결론

- RxJava는 복잡한 비동기 프로그래밍 작업을 처리하는 데 좋은 선택이다.   
- Observable을 중심으로 한 다양한 연산자들을 활용하여 데이터 스트림을 효과적으로 조작하고 변화를 감지할 수 있다.   
- 자바 9의 Flow API와 비교하여 더욱 풍부한 기능과 도구를 제공하며, 시각적 표현 방법을 통해 복잡한 연산을 쉽게 이해할 수 있다.   
- 이러한 특징들은 복잡한 데이터 처리와 분석, 실시간 모니터링 등 다양한 애플리케이션에서 유용하게 활용될 수 있다.
