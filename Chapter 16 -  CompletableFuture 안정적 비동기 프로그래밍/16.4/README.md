# 비동기 작업 파이프라인 만들기

## 할인 서비스 구현

상점에서 제공한 문자열 파싱은 Quote 클래스로 캡슐화하는 코드이다.

```java
public class Quote {
    private final String shopName;
    private final double price;
    private final Discount.Code discountCode;

    public Quote(String shopName, double price, Discount.Code code) {
        this.shopName = shopName;
        this.price = price;
        this.discountCode = code;
    }

    public static Quote parse(String s) {
        String[] split = s.split(":");

        String shopName = split[0];
        double price = Double.parseDouble(split[1]);
        Discount.code discountCode = Discount.Code.valueOf(split[2]);
        return new Quote(shopName, price, discountCode);
    }

    public String getShopName() {
        return shopName;
    }

    public double getPrice() {
        return price;
    }

    public Discount.Code getDiscountCode() {
        return discountCode;
    }
}
```

상점에서 얻은 문자열을 parse로 넘겨주면 상점 이름, 할인 전 가격, 할인된 가격 정보를 포함하는 Quote 클래스 인스턴스가 생성된다.

## 할인 서비스 사용

Discount는 원격 서비스이므로 1초의 지연을 추가한다. 16.3절에서는 가장 쉬운 방법인 순차적 및 동기 방식으로 할인을 적용하는 findPrices 메서드를 구현하였다.

하지만 이러한 구현은 순차적으로 정보를 얻어오는 과정이 존재하므로 성능 최적화된 코드라고 할 수 없다. 그렇다고 하여 병렬 스트림을 이용하면 고정된 스레드 풀의 크기로 인해 검색 대상이 확장되었을 때 유연하게 대응할 수 없게 된다.

이러한 문제를 해결하기 위해서는 CompletableFuture에서 수행하는, 태스크를 설정할 수 있는 커스텀 Executor를 정의함으로써 CPU 사용을 극대화할 수 있다.

## 동기 작업과 비동기 작업 조합하기

아래 코드는 findPrices 메서드를 비동기적으로 구현할 경우 다음과 같은 변환 과정을 거칠 수 있다.

<img width="202" alt="image" src="https://github.com/java-piledrivers/modern-java-in-action/assets/77332981/84d28eab-a1a7-496b-aec1-a39d0975eba4">


1. 가격 정보 얻기: 팩토리 메서드 supplyAsync에 람다 표현식을 전달해서 비동기적으로 상점에서 정보를 조회하고, 작업이 끝났을 때는 해당 상점에서 반환하는 문자열 정보를 포함하게 된다.
  
2. Quote 파싱하기: 첫 번째 단계에서의 문자열을 Quote로 변환한다. 파싱 동작에서는 원격 서비스나 I/O가 없으므로 원하는 즉시 동작을 수행할 수 있다.
  
3. CompleteableFuture를 조합해서 할인된 가격 계산하기: 상점에서 받은 할인 전 가격에 원격 Discount 서비스에서 제공하는 할인율을 적용해야 한다. 이번에는 원격 실행이 포함되므로 이전 두 변환과 다르며 동기적으로 작업을 수행해야 한다.</br></br>'상점에서 가격 정보를 얻어와서 Quote로 변환하기', '변환된 Quote를 Discount 서비스로 전달해서 할인된 최종가격 획득하기', 총 두 개의 비동기 동작을 만들 수 있는데, 자바 8의 CompletableFuture API에서 두 비동기 연산을 파이프라인으로 만들 수 있도록 하는 thenCompose 메서드를 제공한다.
  </br></br>해당 메서드를 이용하면 첫 번째 연산 결과를 두 번째 연산으로 전달하여 연쇄적으로 수행되는 두 개의 비동기 동작을 만들 수 있다.
  

## 독립 CompletableFuture와 비독립 CompletableFuture 합치기

실전에서는 독립으로 실행한 두 개의 CompletableFuture 결과를 합쳐야 하는 상황이 종종 발생하는데, 이때 첫 번째 CompletableFuture의 동작 완료와 관계없이 두 번째 CompletableFuture를 실행할 수 있어야 한다.
이러한 상황에서는 thenCombine 메서드를 사용할 수 있는데, 인자를 통해 두 개의 결과를 어떻게 합칠 것인지 정의할 수 있다.

더불어 thenCompose 메서드와 마찬가지로 thenCombine에도 Async 버전이 존재하는데, 해당 메서드의 경우 정의한 조합 동작이 스레드 풀로 제출되면서 별도의 태스크에서 비동기적으로 수헹된다.

## Future의 리플렉션과 CompletableFuture의 리플렉션

자바 8 이후로 람다 덕분에 다양한 동기 태스크, 비동기 태스크를 활용하여 복잡한 연산 수행 방법을 효과적으로 쉽게 정의할 수 있는 선언형 API를 만들 수 있다.

## 타임아웃 효과적으로 사용하기

Future의 계산 결과를 읽을 때는 무한정 기다리는 상황이 발생할 수 있으므로 블록을 하지 않는 것이 좋은데, 자바 9에서는 CompletableFuture에서 제공하는 몇 가지 기능을 이용해 이러한 문제를 해결할 수 있다.

1. orTimeout: 지정된 시간이 지난 이후에 CompletableFuture를 TimeoutException으로 완료하면서 또 다른 CompletableFuture를 반환할 수 있도록 한다. 해당 메서드를 이용하면 계산 파이프라인을 연결하고 TimeoutException이 발생했을 때 사용자가 쉽게 이해할 수 있는 메시지를 제공할 수 있다.
  
2. completeOnTimeout: CompletableFuture를 반환하므로 결과를 다른 CompletableFuture 메서드와 연결할 수 있다.
