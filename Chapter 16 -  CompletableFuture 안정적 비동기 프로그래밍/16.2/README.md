# 16.2 비동기 API 구현
상품의 최저가격 검색을 위한 애플리케이션을 구현하는 예제이다.<br>
가격 조회 시, 관련 프로모션 등 외부 서비스 접근에 1초 가량 지연 된다고 가정하고, 메서드 delay()로 이를 대체한다.
```java
public double getPrice(String product) {
  return calculatePrice(product);
}

private double calculatePrice(String product) {
  delay(); //사용자가 이 API를 호출 시 비동기 동작 완료까지 1초 블록된다. 
  return random.nextDouble() * product.charAt(0) + product.charAt(1);
}
```

## 16.2.1 동기 메서드를 비동기 메서드로 변환
위 동기 메서드 getPrice를 비동기 메서드 getPriceAsync로 변환한다. 
```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>(); //계산 결과를 포함할 CompletableFuture 생성
  new Thread(() -> {
    double price = calculatePrice(product); //다른 스레드에서 비동기적 계산 수행
    futurePrice.complete(price); //오랜 시간이 걸리는 계산 완료 후, Future에 값 설정
  }).start();
  return futurePrice; //계산 완료를 기다리지 않고 Future 반환
}
```
비동기 계산과 완료 결과를 포함하는 CompletableFuture 인스턴스를 만들었다.<br>
그리고 실제 가격을 계산할 다른 스레드를 만든 후, 계산 결과를 기다리지 않고 결과를 포함할 Future 인스턴트를 바로 반환했다.

```java
Shop shop = new Shop("BestShop");
long start = System.nanoTime();
Future<Double> futurePrice = shop.getPriceAsync("my favorite product"); //상점에 제품가격 요청
long invocationTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Invocation returned after " + inovocationTime + " msecs");

//제품의 가격을 계산하는 동안
doSomethingElse();
//다른 상점 검색 등 작업 수행
try {
  double price = futurePrice.get(); //가격정보가 있으면 Future에서 읽고, 없으면 가격정보를 받을때까지 블록
  System.out.printf("Price is %.2f%n", price);
} catch (Exception e) {
  throw new RuntimeException(e);
}
long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Price returned after " + retrievalTime + " msecs");

//Invocation reeturned after 43 msecs
//Price is 123.26
//Price returned after 1045 msecs
```
가격 계산 API는 비동기로 처리되므로 즉시 Future를 반환하고, 클라이언트는 그 사이에 다른 작업을 처리할 수 있다.<br>
다른작업이 끝나고 클라이언트가 할 일이 없을 때, Future의 get 메서드를 호출해서 가격정보를 받을 때까지 대기한다.

## 16.2.2 에러 처리 방법
가격을 계산하는 동안 예외가 발생하면 해당 스레드에만 영향을 미친다.<br>
즉, 클라이언트는 get 메서드가 반환될 때까지 영원히 기다리게 될 수 있다.<br>
이처럼 블록 문제가 발생할 수 있는 상황에서는 타임아웃을 활용하는 것이 좋다.<br>
그래야 영원히 블록되지 않고 타임아웃 시간이 지나면 TimeoutException을 받을 수 있다.<br>
<br>
계산 도중에 발생된 에러에 대해 알기 위해서는 completeExceptionally 메서드를 이용한다.
```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>();
  new Thread(() -> {
    try {
      double price = calculatePrice(product);
      futurePrice.complete(price);  //계산 정상 종료 시, Future에 가격 정보를 저장하고 종료
    } catch (Exception ex) {
      futurePrice.completeExceptionally(ex); //도중에 문제 발생 시, 발생한 에러를 포함시켜 Future를 종료
    }
  }).start();
  return futurePrice;
}
```
### 팩토리 메서드 supplyAsync로 CompletableFuture 만들기
CompletableFuture를 더 간단하게 만드는 방법도 있다.
```java
public Future<Double> getPriceAsync(String product) {
  return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```
supplyAsync 메서드는 Supplier를 인수로 받아서 CompletableFuture를 반환한다.<br>
ForkJoinPool의 Executor 중 하나가 Supplier를 실행하며,<br> 
두 번째 인수를 받는 오버로드 버전의 supplyAsync 메서드를 이용해 다른 Executor를 지정할 수도 있다.
