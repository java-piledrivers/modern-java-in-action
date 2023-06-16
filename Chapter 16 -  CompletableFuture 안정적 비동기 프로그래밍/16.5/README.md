# 16.5 CompletableFuture의 종료에 대응하는 방법

## 16.5.1 최저가격 검색 애플리케이션 리팩터링
- 모든 가격 정보를 포함할 때까지 리스트 생성을 기다리지 않도록 프로그램을 만들자.
- 그러려면 CompletableFuture의 스트림을 직접 제어해야 한다.

### Future 스트림을 반환하도록 finPrices 메서드 리팩터링

네번째 map 연산을 적용하자. 이 연산은 단순하게 각 CompletableFuture에 등록한다.   
자바 8의 CompletableFuture API는 thenAccept라는 메서드로 이 기능을 제공한다.

- thenAccept : 계산이 끝나면 값을 소비한다.(Consumer를 인수로 받음)
- thenAcceptAsync : CompletableFuture가 완료된 스레드가 아니라 새로운 스레드를 이용해서 값을 소비한다.
- allOf : 전달받은 CompletableFuture 배열이 모두 완료될 때 CompletableFuture<Void>를 리턴한다.

이 경우에는 즉시 응답해야 하므로 thenAccept를 사용한다. 오히려 thenAcceptAsync를 사용하면 새로운 스레드를 이용할 수 있을 때까지 기다려야 할 수 있다.

```java
@Test
public void findPricesStream() {
    long start = System.nanoTime();
    findPricesStream("myPhone").map(f -> f.thenAccept(System.out::println));
    long duration = (System.nanoTime() - start) / 1_000_000;
    CompletableFuture[] futures = findPricesStream("myPhone").map(f -> f.thenAccept(System.out::println)).toArray(size -> new CompletableFuture[size]);
    CompletableFuture.allOf(futures).join();
    log.info("time : {}", duration);
    }
private Stream<CompletableFuture<String>> findPricesStream(String product) {
    return shops.stream()
    .map(shop -> CompletableFuture.supplyAsync(() -> shop.getPriceRandomDelayed(product), executor))
    .map(future -> future.thenApply(Quote::parse))
    .map(future -> future.thenCompose(quote -> CompletableFuture.supplyAsync(() -> Discount.applyDiscount(quote), executor)));
    }
```

다음과 같은 결과가 나온다
```
BuyItAll price is 191.259
BestPrice price is 203.4995
LetsSaveBig price is 102.648
MyFavoriteShop price is 175.294
06:53:17.729 [Test worker] INFO com.example.javainactionpractice.JavaInActionPracticeApplicationTests -- time : 3214
```

## 16.5.2 응용
어떤 부분이 달라졌는지 좀 더 명확하게 확인할 수 있도록 각각의 계산에 소요된 시간을 출력하는 부분을 코드에 추가했다.

```java
@Test
public void findPricesStreamApply() {
    long start = System.nanoTime();
    
    CompletableFuture[] futures = findPricesStream("myPhone")
    .map(f -> f.thenAccept(s -> log.info("{} done in {} msecs", s, (System.nanoTime() - start) / 1_000_000)))
    .toArray(size -> new CompletableFuture[size]);
    
    CompletableFuture.allOf(futures).join();
    
    long duration = (System.nanoTime() - start) / 1_000_000;
    log.info("All shops have now responded in = {}", duration);
```

다음과 같은 결과가 나온다
```
06:51:25.317 [pool-1-thread-5] INFO com.example.javainactionpractice.JavaInActionPracticeApplicationTests -- MyFavoriteShop price is 142.816 done in 1897 msecs
06:51:26.017 [pool-1-thread-3] INFO com.example.javainactionpractice.JavaInActionPracticeApplicationTests -- BuyItAll price is 222.12 done in 2599 msecs
06:51:26.173 [pool-1-thread-4] INFO com.example.javainactionpractice.JavaInActionPracticeApplicationTests -- BestPrice price is 182.38 done in 2755 msecs
06:51:26.755 [pool-1-thread-5] INFO com.example.javainactionpractice.JavaInActionPracticeApplicationTests -- LetsSaveBig price is 137.81699999999998 done in 3337 msecs
06:51:26.756 [Test worker] INFO com.example.javainactionpractice.JavaInActionPracticeApplicationTests -- All shops have now responded in = 3337
```

임의의 지연이 추가되면 마지막 가격 정보에 비해 처음 가격 정보를 두 배 빨리 얻는다는 것을 출력 결과에서 확인할 수 있다.
