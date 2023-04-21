# Spliterator 인터페이스

Spliterator는 분할할 수 있는 반복자라는 의미이다.   
Iterator 처럼 Spliterator는 소스의 요소 탐색 기능을 제공한다는 점은 같지만 Spliterator는 병렬 작업에 특화되어 있다.   
커스텀 Spliterator를 꼭 구현해야 하는 건 아니지만 Spliterator가 어떻게 동작하는지 이해한다면 병렬 스트림 동작과 관련한 통찰력을 얻을 수 있다.

자바 8은 컬렉션 프레임워크에 포함된 모든 자료구조에서 사용할 수 있는 디폴드 Spliterator 구현을 제공한다.

```java
public interface Spliterator<T> {

  boolean tryAdvance(Consumer<? super T> action);

  default void forEachRemaining(Consumer<T> action);

  Spliterator<T> trySplit();

  long estimateSize();

  int characteristics();
}
```

### tryAdvance

- 요소를 하나씩 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 true 리턴

### forEachRemaining

- 더이상 splitting이 필요하지 않을때 특정 action을 하는 함수이다.
- 기본적으로 각 남아있는 Element에 따라 action을 취할 수 있고, 다 처리될 때 까지 현재 스레드에서 sequential하게 처리된다
- 디폴트로 구현된 메소드를 살펴보면 번복적으로 tryAdvance를 호출하여, spliterator 원소를 시퀀셜 하게 처리한다.
- spliting을 수행하는 동안, spiliterator는 시퀀셜하게 처리해도 될 정도로 원소의 수가 줄어들면, forEachRemaing 메소드를 통해 시퀀셜 하게 처리한다.

```java
default void forEachRemaining(Consumer<T> action){
    do{

    }while(tryAdvance(action));
    }
```

### trySplit

- 일부 요소를 두 개의 Spliterator로 분할해서 병렬로 처리할 수 있도록 함

### estimateSize

- 탐색해야 할 요소 수 정보를 제공

### characteristics

- Spliterator 자체의 특성 값을 반환

## Spliterator

```java
public class WordCounterSpliterator implements Spliterator<Character> {

  private final String string;
  private int currentChar = 0;

  public WordCounter(String string) {
    this.string = string;
  }

  // 탐색 할 요소가 남아있다면 true 반환
  @Override
  public boolean tryAdvance(Consumer<? super Character> action) {
    action.accept(string.charAt(currentChar++)); // 현재 문자열을 소비
    return currentChar < string.length(); // 소비할 문자가 남아있으면 true 반환
  }

  // 요소를 분할해서 Spliterator 생성
  @Override
  public Spliterator<Character> trySplit() {
    int currentSize = string.length() - currentChar;
    if (currentSize < 10)
      return null;

    // 파싱할 문자열의 중간을 분할 위치로 설정
    for (int splitPos = currentSize / 2 + currentChar; splitPos < string.length(); splitPos++) {
      if (Character.isWhitespace(string.charAt(splitPos))) { // 공백 문자가 나올때
        // 문자열을 분할 해 Spliterator 생성
        Spliterator<Character> spliterator = new WordCounter(
            string.substring(currentChar, splitPos));
        // 시작을 분할 위치로 설정
        currentChar = splitPos;
        return spliterator;
      }
    }
    return null;
  }

  // 탐색해야 할 요소의 수
  @Override
  public long estimateSize() {
    return string.length() - currentChar;
  }

  // Spliterator 객체에 포함된 모든 특성값의 합을 반환
  @Override
  public int characteristics() {
    // ORDERED : 문자열의 순서가 유의미함
    // SIZED : estimatedSize 메서드의 반환값이 정확함
    // NONNULL : 문자열에는 null이 존재하지 않음
    // IMMUTABLE : 문자열 자체가 불변 클래스이므로 파싱하며 속성이 추가되지 않음
    return ORDERED + SIZED + SUBSIZED + NONNULL + IMMUTABLE;
  }

}
```

### Spliterator Workflow

![parallel_proc_1.png](https://java-8-tips.readthedocs.io/en/stable/_images/parallel_proc_1.png)
출처: https://java-8-tips.readthedocs.io/en/stable/parallelization.html#spliterator

# Reference

https://okky.kr/articles/345720
