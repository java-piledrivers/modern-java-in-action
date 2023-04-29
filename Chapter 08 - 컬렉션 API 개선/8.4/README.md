# 8.4 개선된 ConcurrentHashMap

ConcurrentHashMap 클래스는 동시성 친화적이며 최신 기술을 반영한 HashMap 버전이다. 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.  
떄문에 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다.  


## 8.4.1 리듀스와 검색


ConcurrentHashMap은 스트림에서 봤던 녀석들과 비슷한 종류의 세 가지 새로운 연산을 제공한다.

* forEach: 각 (키, 값) 쌍에 주어진 액션을 실행
* reduce: 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
* search: 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용  

또 키, 값 그리고 Map.Entry 를 활용해서도 연산을 수행할 수 있는 메서드도 있다.

* 키, 값으로 연산(forEach, reduce, search)
* 키로 연산(forEachKey, reduceKeys, searchKeys)
* 값으로 연산(forEachValue, reduceValues, searchValues)
* Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)


아래의 예제는 reduceVlues 메서드를 사용하여 맵의 최대값을 찾는다.

```java
    ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
    long parallelismThreshold = 1;
    Optional<Long> maxValue = Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```


## 8.4.2 계수

ConcurrentHashMap 클래스에서는 맵의 매핑 개수를 알고싶을때에는 mappingCount 메서드를 사용하는 것이 좋다. 기존의 size 메서드는 int 를 반환하기 때문에 Long을 반환하는 mappingCount 메서드를 사용하여 오버플로우를 방지하자.


## 8.4.3 집합뷰

ConcurrentHashMap 클래스를 집합뷰로 반환하는 keySet 메서드도 제공한다. 이때, 맵을 바꾸면 집합도 바뀌고, 반대로 집합을 변경하면 맵도 영향을 받는다. 


---


## 번외 HashTable v.s HashMap v.s ConcurrentHashMap


3가지 모두 <Key, Value> 형식으로 매핑된다는 공통점은 있지만 각각의 차이점들을 갖고 있다. 


### HashMap 

* key, value 에 null 을 허용한다.
* 동기화를 보장하지 않는다.


### HashTable

* key, value 에 null 을 허용하지 않는다.
* 동기화를 보장한다. 


### ConcurrentHashMap
* key, value 에 null 을 허용하지 않는다.
* 동기화를 보장한다. 


여기 까지 봤을 떄 HashMap 과 다른 Map들의 차이점은 알겠는데, HashTable과 ConcurrentHashMap 의 차이점이 궁금할 수 있다.  
이 둘의 차이점은 앞서 설명했던 것처럼 읽기 쓰기 연산 성능에서 ConcurrentHashMap 이 HashTable 보다 더 월등하다.  

HashTable은 thread-safe를 유지하기 위해 데이터를 다루는 메서드에 *synchronized* 가 붙어있어 해당 메서드 실행시마다 동기화 락을 건다. 여기서 이 동기화 락이 매우 느리다는 단점이 있다.   
반면 ConcurrentHashMap 동기화 처리를 할 때, 어떤 Entry를 조작하는 경우에 해당 Entry에 대해서만 락을 건다. 그래서 HashTable보다 데이터를 다루는 속도가 빠르다. 즉, Entry 아이템별로 락을 걸어 멀티 쓰레드 환경에서의 성능을 향상시킨다.


##### ref: [HashMap vs HashTable vs ConcurrentHashMap](https://tecoble.techcourse.co.kr/post/2021-11-26-hashmap-hashtable-concurrenthashmap/)
