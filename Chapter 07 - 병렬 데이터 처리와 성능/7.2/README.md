# 포크/조인 프레임워크
포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었음.  
해당 프레임워크에서는 서브태스크를 스레드 풀의 작업자 스레드에 분할 할당하는 ExecutorService 인터페이스를 구현

## RecursiveTask 활용

스레드 풀을 이용하려면 RecursiveTask의 서브 클래스를 만들어야 하며, RecursiveTask를 정의하기 위해서는 추상 메서드 compute를 구현해야 한다.

compute 메서드는 다음과 같은 알고리즘을 정의한다.

1.  태스크를 서브태스크로 분할
2.  더 이상 분할할 수 없을 때 개별 서브태스크의 결과 생산

위와 같은 알고리즘을 아래와 같은 의사코드 형식으로 표현할 수 있다.

```java
if (태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
    순차적으로 태스크 계산
} else {
    태스크를 두 서브태스크로 분할
    태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출
    각 서브태스크의 결과를 합침
}
```

포크/조인 프레임워크를 이용해서 병렬 합계를 수행하는 예시 코드이다.

```java
//포크/조인 프레임워크에서 사용할 태스크 생성
public class ForkJoinSumCalculator extends jsva.util.concurrent.RecursiveTask<Long> {
    private final long[] numbers;
    private final int start;
    private final int end;
    public static final long THRESHOLD = 10_000; //분할 가능한 서브태스크의 최소 크기

    public ForkJoinSumCalculator(long[] numbers) { //메인 태스크를 생성할 때 사용할 공개 생성자
        this(numbers, 0, numbers.length);
    }

    private ForkJoinSumCalculator(long[] numbers, int start, int end) { //서브태스크를 재귀적으로 만들 때 사용할 비공개 생성자
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= THRESHOLD) {
            return computeSequentially();
        }

        //서브태스크 생성 -> 첫 번째 절반 합산
        ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
        leftTask.fork();

        //서브태스크 생성 -> 나머지 절반 합산
        ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
        Long rightResult = rightTask.compute();
        Long leftResult = leftTask.join();
        return leftResult + rightResult;

        //더 분할할 수 없을 때 서브태스크의 결과를 계산하는 알고리즘
        private long computeSequentially() {
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += numbers[i];
            }
            return sum;
        }
    }
}
```

## 포크/조인 프레임워크를 제대로 사용하는 방법

-   join 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록시키기 때문에, **두 서브태스크가 모두 시작된 다음에 join을 호출**해야 한 서브태스크가 서로 다른 서브태스크의 종료를 기다리지 않게 된다. 이러한 과정은 원래 순차 알고리즘보다 느리고 복잡한 프로그램을 야기할 수 있다.
-   RecursiveTask 내에서는 ForkJoinPool의 invoke 메서드를 사용하지 말고 **compute나 fork 메서드를 직접 호출**하자. invoke는 순차 코드에서 병렬 계산을 시작할 때만 사용하도록 한다.
-   두 서브태스크에 모두 fork 메서드를 호출하는 것보다는 **다른 한쪽에 compute를 호출**하는 것이 효율적이다. 이렇게 하면 두 서브태스크의 한 태스크에는 같은 스레드를 재사용할 수 있으므로 풀에서 불필요한 태스크를 할당하는 오버헤드를 피할 수 있다.
-   보통의 디버깅 상황은 스택 트레이스를 통해 문제가 일어난 과정을 쉽게 확인할 수 있는 반면 포크/조인 프레임워크에서는 fork를 통해 다른 스레드에서 compute를 호출하므로 **스택 트레이스가 크게 도움이 되지 않는다.**
-   **멀티코어에 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠르지는 않다.** 병렬 처리로 성능 개선을 기대하기 위해서는 태스크를 여러 독립적인 서브태스크로 분할할 수 있어야 하며, 각 서브태스크의 실행시간은 새로운 태스크를 포기하는 데 드는 시간보다 길어야 한다.
-   컴파일러 최적화는 병렬 버전보다는 **순차 버전에 집중될 수도 있다.** (e.g. 죽은 코드를 분석해서 분할 여부 결정, 사용하지 않는 죽은 코드 폐기)

## 작업 훔치기

이론적으로는 코어 개수만큼 병렬화된 태스크로 작업부하를 분할하면 모든 CPU 코어에서 태스크를 실행할 것이고, 크기가 같은 각각의 태스크는 같은 시간에 종료될 것이라 생각할 수 있다. 하지만 복잡한 시나리오의 경우 각각의 서브태스크의 작업완료 시간이 크게 달라질 수 있다.  
분할 기법이 효율적이지 않았기 때문일 수도 있고 디스크 접근 속도가 저하되었거나 외부 서비스와 협력하는 과정에서 지연이 생길 수 있기 때문이다.

포크/조인 프레임워크에서는 작업 훔치기라는 기법으로 이 문제를 해결하는데, 각각의 스레드는 **자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조**하면서 작업이 끝날 때마다 **큐의 헤드에서 다른 태스크를 가져와서** 작업을 처리한다. 이러한 과정은 모든 큐가 빌 때까지 반복된다.
