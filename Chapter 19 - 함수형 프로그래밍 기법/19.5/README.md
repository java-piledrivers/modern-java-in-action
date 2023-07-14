
# 5. 기타 정보

참조 투명성 : 함수 (또는 메쏘드) 가 함수 외부의 영향을 받지 않는 것

# 1. 캐싱 또는 기억화

기억화는 메서드에 래퍼로 캐시를 추가하는 기법

래퍼가 호출되면 캐시에 존재하는지 먼저 확인하고 있으면 리턴, 없으면 계산

다수의 호출자가 공유하는 자료구조를 갱신하기 때문에 순수 함수형 해결방식은 아니다

그러나 감싼 버전의 코드는 참조 투명성을 유지할 수 있다.

```java

@Slf4j
public class Ch19FunctionalProgrammingAdv {
@Test
public void cachingMemorization() {
  Integer result1 = computeNumberOfNodesUsingCache("1");
  log.info("result 1 :{}", result1);
  Integer result2 = computeNumberOfNodesUsingCache("1");
  log.info("result 2 :{}", result2);
}
private static final Map<String, Integer> numberOfNodes = new HashMap();
Integer computeNumberOfNodesUsingCache(String id) {
  Integer result = numberOfNodes.get(id);
  if (result != null) {
    return result;
  }
  result = compute();
  numberOfNodes.put(id, result);
  return result;
}
private static Integer compute() {
  try {
    Thread.sleep(1000L);
  } catch (Exception e) {

  }
  return 10;
}
```


# 2.콤비네이터

함수형 프로그래밍에서는 두 함수를 인수로 받아 다른 함수를 반환하는 등 함수를 조합하는 고차원 함수를 많이 사용하게 된다.

함수를 조합하는 기능을 콤비네이터라고 한다.


```java

public void combineFunctionsTest() {
  Integer result = Combinators
      // x -> (2 * (2 * (2 * x) ) ) 또는 x -> 8*x
      .repeat(3, (Integer x) -> 2 * x)
      .apply(10);
  log.info("result :{}", result);
}
}
public class Combinators {
// 함수 f와 g를 인수로 받아 f의 기능을 적용한 다음 g의 기능을 적용하는 함수를 반환
public static <A, B, C> Function<A, C> compose(Function<B, C> g, Function<A, B> f) {
  return x -> g.apply(f.apply(x));
}
// f에 연속적으로 n번 적용
public static <A> Function<A, A> repeat(int n, Function<A, A> f) {
  return n == 0 ? x -> x : compose(f, repeat(n - 1, f));
}
}

```
