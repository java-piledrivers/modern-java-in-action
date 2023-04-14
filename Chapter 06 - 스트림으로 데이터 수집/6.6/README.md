# 6.6 커스텀 컬렉터를 구현해서 성능 개선하기
```java
public static Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n).boxed()
                    .collect(partitioningBy(candidate -> isPrime(candidate)));
}
```
2에서 n까지의 수(rangeClosed)를 <br/>
소수인지 아닌지(isPrime) 여부로<br/>
분할하는 메서드<br/><br/>

```java
private static boolean isPrime(Integer candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);

    return IntStream.rangeClosed(2, candidateRoot)
                    .noneMatch(i -> candidate % i == 0);
}
```
2에서 대상의 제곱근(candidateRoot)까지의 수 중<br/>
대상이 나누어떨어지는 수가 없는 경우(noneMatch)는 true 소수이다, <br/>
있는 경우는 false 소수가 아니다를 반환하는 프레디케이트<br/><br/>

isPrime이 대상의 제곱근까지만 체크하는 이유?<br/>
소수 여부를 검사할 수에 대해서<br/>
그 값의 제곱근을 기준으로 대칭적으로 곱이 일어나므로 <br/>
제곱근 이하의 값까지만 검사를 하면 나머지는 검사를 할 필요가 없다.<br/>
![image](https://user-images.githubusercontent.com/50727201/232108019-2675de44-bbdb-4468-a030-5c3c5adb9ce3.png)

## 6.6.1 소수로만 나누기
### 1. predicate 구현하기
제수(devisor)가 소수가 아니면 소용 없으므로, 제수를 '현재 숫자 이하에서 발견한 소수'로 제한할 수 있다. <br/>
따라서 중간결과 리스트가 있다면, isPrime 메서드의 파라미터로 넘겨 개선할 수 있다. 
```java
public static boolean isPrime(List<Integer> primes, Integer candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return primes.stream()
                 .takeWhile(i -> i <= candidateRoot)
                 .noneMatch(i -> candidate % i == 0);
}
```
위의 숫자형 스트림에서 rangeClosed를 이용해 대상의 제곱근 이하의 값까지로만 범위를 제한한 것처럼, <br/>
takeWhile의 쇼트서킷을 이용해 제곱근 이하의 값까지만 체크한다. <br/><br/>
### 2. Custom Collector 구현하기
6장 5절의 다섯 가지 메서드를 이용
```java
public class PrimNumberCollector implements Collector<Integer, Map<Boolean, List<Integer>>, Map<Boolean, List<Integer>>> {

    /**
     * 수집과정의 중간결과, 즉 지금까지 발견한 소수를 포함하는 누적자로 사용할 맵을 만들고,
     * 소수와 비소수를 수집하는 두 개의 리스트를 각각 true, false 키와 빈 리스트로 초기화 한다. (p226)
     */
    @Override
    public Supplier<Map<Boolean, List<Integer>>> supplier() {
        return () -> new HashMap<>() {{
            put(true, new ArrayList<>());
            put(false, new ArrayList<>());
        }};
    }

    /**
     * 스트림의 요소를 수집한다. (p226)
     * 누적 리스트 acc에서 isPrime 메서드 결과에 따라 소수(true), 비소수(false) 리스트에 candidate를 추가 한다.
     */
    @Override
    public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
        return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
           acc.get( isPrime( acc.get(true), //지금까지 발견한 소수 리스트(supplier true 키 분할 리스트)를 isPrime으로 전달
               candidate) )
              .add(candidate);  //isPrime 메서드의 결과에 따라 맵에서 알맞은 리스트를 받아 현재 candidate를 추가
        };
    }

    private static boolean isPrime(List<Integer> primes, int candidate) {
        int candidateRoot = (int) Math.sqrt((double) candidate);
        return primes.stream()
                     .takeWhile(i -> i <= candidateRoot)
                     .noneMatch(i -> candidate % i == 0);
    }

    /**
     * 알고리즘이 순차적이기에 실제 병렬로 사용할 수는 없지만, 학습용으로 만든 메서드. 여기선 실제 사용되지 않는다.
     * 각 스레드에서 만든 누적 결과를 병합하여 반환한다. (p228)
     */
    @Override
    public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
        return (Map<Boolean, List<Integer>> map1, 
            Map<Boolean, List<Integer>> map2) -> {
                map1.get(true).addAll(map2.get(true)); //두 번째 맵의 소수 리스트를 첫 번째 맵의 소수 리스트에 추가
                map1.get(false).addAll(map2.get(false)); //두 번째 맵의 비소수 리스트를 첫 번째 맵의 비소수 리스트에 추가
            return map1;
        };
    }

    /**
     * 본래 누적자를 최종결과로 변환하는 역할을 하지만, (p227)
     * 여기선 누적자인 accumulator가 이미 최종 결과와 같으므로 변환 과정이 필요 없어 항등 함수를 반환하도록 구현한다.
     */
    @Override
    public Function<Map<Boolean, List<Integer>>, Map<Boolean, List<Integer>>> finisher() {
        return Function.identity();
    }

    /**
     * 발견한 소수의 순서에 의미가 있으므로 CONCURRENT, UNORDERED는 해당되지 않고, IDENTITY_FINISH만 설정한다.
     * 추후 최적화를 위한 힌트 제공 (p229)
     */
    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));
    }
}
```
<br/>

### 컬렉터 인터페이스를 별도 구현 없이 사용 
코드는 간결하지만, 가독성 및 재사용성은 떨어짐<br/>

```java
static Map<Boolean, List<Integer>> partitionPrimesWithCustomCollectorV2(int n) {
    return IntStream.rangeClosed(2, n).boxed()
            .collect(
                    () -> new HashMap<Boolean, List<Integer>>() {{    //발행
                        put(true, new ArrayList<>());
                        put(false, new ArrayList<>());
                    }},
                    (acc, candidate) -> {    //누적
                        acc.get(isPrime(acc.get(true), candidate)).add(candidate);
                    },
                    (map1, map2) -> {   //병합
                        map1.get(true).addAll(map2.get(true));
                        map1.get(false).addAll(map2.get(false));
                    }
            );
}
```
