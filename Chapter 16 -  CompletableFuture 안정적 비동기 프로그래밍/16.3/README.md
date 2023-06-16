 # 16.3 비블록(non-block) 코드 만들기

요약: 스트림 -> 병렬 스트림 -> CompletableFuture 활용하여 병렬 작업을 개선할 수 있다.

예제코드: https://github.com/kyupid/java-test/tree/main/modern-chapter16/src/main/java/org/example

## 테스트 코드

```java
    public static void test(List<Shop> shops) {
        long start = System.nanoTime();
        System.out.println(findPricesV1(shops, "myPhone27S"));
        long duration = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Done in " + duration + " msecs");
    }
```

## 스트림 버전

shop.getPrice 는 임의로 price 를 찾을때마다 1초정도 지연시켰다는 것을 인지하고 아래를 보자.

```java
    private static List<String> findPrices(List<Shop> shops, String product) {
        return shops.stream()
                .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }


```
```
결과

[BestPrice price is 203.01, LetsSaveBig price is 188.42, MyFavoriteShop price is 203.86, BuyItAll price is 126.32]
Done in 4048 msecs
```

순차적으로 실행했기 때문에 하나에 1초 총 4개를 돌렸으므로 약 4초정도의 결과가 나왔다.

## 병렬 스트림 버전

```java
    private static List<String> findPrices(List<Shop> shops, String product) {
        return shops.parallelStream()
                .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }
```
```
결과

[BestPrice price is 218.33, LetsSaveBig price is 160.23, MyFavoriteShop price is 220.10, BuyItAll price is 193.95]
Done in 1053 msecs
```

병렬로 실행해서 1초의 결과가 나와서 개선되었다. 이를 더 개선해보자.

## CompletableFuture 버전

```java
    private static List<String> findPrices(List<Shop> shops, String product) {
        List<CompletableFuture<String>> priceFutures = shops.stream()
                .map(shop -> CompletableFuture.supplyAsync(() -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))))
                .collect(Collectors.toList());

        return priceFutures.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
    }
```
```
결과

[BestPrice price is 172.50, LetsSaveBig price is 193.90, MyFavoriteShop price is 142.92, BuyItAll price is 152.91]
Done in 2069 msecs
```
2초 라는 결과가 나왔지만, Shop 이 많아질수록 CompletableFuture 의 효과는 커진다. 왜 그럴까?

CompletableFuture.supplyAsync() 메소드는 ForkJoinPool.commonPool()을 사용하는데, 이는 JVM에 의해 관리되며 기본적으로 시스템의 코어 수에 따라 스레드 수가 결정된다.

따라서, 스레드 풀 크기와 CPU 코어의 수, 그리고 JVM에 의해 어떻게 스케줄링되는지에 따라 결과가 다르게 나타날 수 있다.

스레드 수를 증가시키는 것이 무조건적으로 성능 향상을 의미하진 않는다. 스레드를 관리하고 컨텍스트 스위칭을 하는데는 비용이 들기 때문에 너무 많은 수의 스레드는 오히려 성능을 저하시킬 수 있다. 따라서 병렬 처리를 위한 최적의 스레드 수를 결정하는 것이 중요한 고려사항이다.

## CompletableFuture 개선 위한 커스텀 Executor 사용

스레드 풀 크기를 조절하는 것에는 "자바 병렬 프로그래밍"에 최적값을 찾는 방법이 알려져있는 방법인것같다.

`N_threads = N_cpu * U_cpu * (1 + W/C)`

- N_threads: 최적의 스레드 풀 크기
- N_cpu: CPU 코어의 수,
- U_cpu: CPU 사용률(0 <= U_cpu <= 1),
- W/C: 대기 시간과 계산 시간의 비율

예를 들어, Shop 의 응답을 대략 99% 시간만큼 기다리미르 W/C 비율을 100으로 간주한다.   
N_cpu가 4라면, 400스레드를 갖는 풀을 만들어야함을 의미한다.   
하지만 상점 수보다 많은 스레드를 가지는것이 의미가없으므로 최대 개수를 100 이하로 설정하는것이 바람직하다. (책에 의하면)

```java
    private static List<String> findPricesV4(List<Shop> shops, String product) {
        final Executor executor = Executors.newFixedThreadPool(Math.min(shops.size(), 100), new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                t.setDaemon(true); // 프로그램 종료를 방해하지 않는 데몬 스레드를 사용한다.
                return t;
            }
        });

        List<CompletableFuture<String>> priceFutures = shops.stream()
                .map(shop -> CompletableFuture.supplyAsync(() -> shop.getName() + " price is " + shop.getPrice(product), executor))
                .collect(Collectors.toList());
        return priceFutures.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
    }
```
```
결과

[BestPrice price is 196.50406616365115, LetsSaveBig price is 184.24541442317607, MyFavoriteShop price is 161.91241143763202, BuyItAll price is 229.4755925617906]
Done in 1058 msecs
```

5개 상점을 검색할땐 1021 밀리초, 9개 상점을 검색할땐 1022 밀리초가 소요된다. 이전에 계산한거처럼 400개의 상점까지 같은 성능을 유지할 수 있다.   
결국 어플리케이션 특성에 맞는 Executor 를 만들어 CompletableFuture 를 활용하는것이 가장 바람직하다는 사실을 확인할 수 있다.   
비동기 동작을 많이 사용하는 상황이라면 지금 기법이 가장 효과적이라는것을 기억하자.   
