# 6-4 분할

분할 함수(partitioning function)라고 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다.
분함 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean이다. 결과적으로 그룹화 맵은 최대 두개의 그룹으로 나뉜다.


```java
/* 분할 함수를 사용하는 예시, 
메뉴에서 모든 요리를 채식 요리와 채식이 아닌 요리를 분류함*/
Map<Boolean, List<Dish>> partitionedMenu =
    menu.stream().collect(partitioningBy(Dish::isVegetarian));

```
해당 코드는 다음과 같은 결과를 반환한다.

{false=[pork, beef, chicken, prawns, salmon],
  true = [french fries, rice, season fruit, pizza]}
  

맵의 키로써 true, false가 들어간걸 확인할 수 있다.

// 참값의 키로 맵에서 value를 얻을 수 있다.
List<Dish> vegetarianDishs = partitionedMenu.get(true);


---

위와 같은 결과는 프레디케이트로 필터링한 다음, 별도의 리스트에 결과를 수집해도 같은 결과를 얻을 수 있다.
  
```java
  
 List<Dish> vegetarianDishes = 
   menu.stream().filter(Dish::isVegetarian).collect(toList());
```
그러나 분할를 사용한다면 스트림 작업을 한 번만 수행하여 결과를 두 개의 그룹으로 분류할 수 있어 가독성 면에서 더 뛰어나다.
또한 두번째 인수를 전달할 수 있는 partitioningBy 메서드를 사용한다면 더욱 세부적으로 리스트를 생성할 수 있다.
  
```java
Map<Boolean, Map<Dish.Type, List<Dish>> vegetarianDishesByType = menu.stream().collect(
      partitioningBy(Dish::isVegetarian, groupingBy(Dish::getType))); 
```
해당 코드를 실행하면 아래와 같은 결과을 얻을 수 있다.
{false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]},
  true={OTHER=[french fries, rice, season fruit, pizza]}}}
  
partitioningBy가 반환한 맵 구현은 코드도 짧아지지만 참과 거짓 두 가지 키만 포함하여, 간결하고 효과적으로 표현할 수 있다.
또한 다수준으로 분할 역시 가능하다.
  
다수준 분할 예제
  숫자를 소수와 비소수로 분할하기
```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class PrimePartitionExample {
    public static void main(String[] args) {
        Map<Boolean, List<Integer>> partitionedNumbers = partitionPrimes(20);
        System.out.println(partitionedNumbers);
    }

    public static Map<Boolean, List<Integer>> partitionPrimes(int n) {
        return IntStream.rangeClosed(2, n).boxed().collect(
            Collectors.partitioningBy(candidate -> isPrime(candidate)));
    }

    // 프레디 케아트 
    public static boolean isPrime(int number) {
        if (number <= 1) {
            return false;
        }
        return IntStream.range(2, number).noneMatch(i -> number % i == 0);
    }
}

  
```
