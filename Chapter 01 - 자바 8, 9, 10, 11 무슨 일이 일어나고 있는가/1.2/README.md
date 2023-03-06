## 1.2 왜 아직도 자바는 변화하는가?

언어는 마치 생태계와 닮았다. 진화하는 언어는 생존하고 그렇지 않은 언어는 도태되어 사장된다.   
자바는 어떻게 성공을 거두었나?

### 프로그래밍 언어 생태계에서 자바의 위치

많은 유용한 라이브러리를 포함하여 잘 설계된 객체지향 언어로 시작하였고,   
스레드와 락을 이용한 소소한 동시성도 지원함.

코드를 JVM 바이트 코드로 컴파일하는 특징때문에 처음에 인터넷 애플릿 프로그램의 주요 언어가 됨

> 애플릿 프로그램이란?
> 인터넷 애플릿은 웹 브라우저에서 실행되는 Java 가상 머신(JVM)을 사용하여 작동.   
> 이를 통해 사용자는 브라우저에서 자바 프로그램을 실행할 수 있으며,   
> 인터넷 애플릿을 사용하여 그래픽, 애니메이션, 데이터베이스 연결 등을 포함한 다양한 작업을 수행 가능.
> 최근에는 보안이슈와 웹 표준 발전으로 사용하지 않음

그리고 자바가 처음에 각광받은 것은 캡슐화와 객체지향 덕분.   
처음에는 C/C++ 비해서 앱을 실행하는데 추가적으로 드는 비용떄문에 자바에 대한 반감이 있었지만,   
점차 하드웨어 발전으로 프로그래머의 시간이 더욱 중요한 요소로 부각 되어짐.

자바 8 은 현재 언어 생태계에서 요구하는 기능을 효과적으로 제공한다.

### 스트림 처리

스트림이란 일련의 연속적으로 구성된 데이터 묶음이라고 보면 된다.   
따라서 스트림 처리란 이런 데이터 흐름들의 처리를 말하는 것이다.   
유닉스 명령어를 `|` 파이프를 이용해 연결하는 것도 스트림 처리이다.

리눅스 명령어 예시: `cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3`

자바8에서 제공하는 스트림 API 가 이런 파이프라인을 만드는데 필요한 다양한 메소드를 제공한다고 생각하면 된다.   
스트림 API의 핵심은 기존엔 한번에 한항목을 처리하던 것을 고수준으로 추상화하여 일련의 스트림으로 만들어 처리가 가능하다는 것.   
또한 여러 CPU 코어를 쉽게 할당할 수 있고, 병렬성을 얻을 수 있다.    

### 동작 파라미터화로 메서드에 코드 전달

위 리눅스 명렁어 예시에서 예를 들어 sort 에 어떤 기준을 넣어 sorting 하고 싶을 수도 있다.   
자바8 이전에는 이를 수행하려면 1.1에서 본것처럼 Comparator 객체를 만들어 sort 에 넘겨주어야하지만 이는 너무 복잡하다
자바8 에서는 이런 기능을 제공해준다.   

이런 기능을 동작. 파라미터화(beavior parameterization)이라고 하는데,   
스트림 API 에서 연산의 동작을 파라미터화할 수 있는 코드를 전달한다는 사상에 기ㅊ초하기 때문에 중요하다

### 병렬성과 공유 가변 데이터

스트림 API 를 통해서 병렬성을 쉽게 얻을 수 있다.   
다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만드려면 공유된 가변 데이터에 접근하지 않아야 한다.

지금까지는(자바 8이전까지는) 독립적으로 실행 될 수 있는 다중 코드 사본과 관련된 병렬성을 고려했다.

> 다중 코드 사본 (multiple code copies) 를 이용한 sum 예제

```java
public class ParallelArraySum {
    private int[] array;
    private int numOfThreads;

    public ParallelArraySum(int[] array, int numOfThreads) {
        this.array = array;
        this.numOfThreads = numOfThreads;
    }

    public int sum() throws InterruptedException {
        int size = (int) Math.ceil(array.length * 1.0 / numOfThreads);
        int[] sums = new int[numOfThreads];
        Thread[] threads = new Thread[numOfThreads];

        for (int i = 0; i < numOfThreads; i++) {
            final int start = i * size;
            final int end = (i + 1) * size;
            threads[i] = new Thread(() -> {
                int sum = 0;
                for (int j = start; j < end && j < array.length; j++) {
                    sum += array[j];
                }
                sums[i] = sum;
            });
            threads[i].start();
        }

        for (int i = 0; i < numOfThreads; i++) {
            threads[i].join();
        }

        int sum = 0;
        for (int i = 0; i < numOfThreads; i++) {
            sum += sums[i];
        }

        return sum;
    }

    public static void main(String[] args) throws InterruptedException {
        int[] array = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        ParallelArraySum parallelArraySum = new ParallelArraySum(array, 4);
        int sum = parallelArraySum.sum();
        System.out.println("Sum: " + sum);
    }
}
```
다중 코드 사본이란 하나의 프로그램에서 여러 개의 복사본을 만들어 각각의 복사본이 데이터를 처리하도록 하는 것이다.   
각각의 threads[i] 들은 똑같은 함수를 구현한 Thread 를 받고 start 하게 되는데, 이를 다중 코드 사본이라고 하는 것이다.

하지만 이런 코드는 공유된 변수나 객체가 있으면 병렬성에 문제가 발생한다

기존처럼 synchronized 를 이용해 공유 가변 데이터를 보호하는 규칙을 만들수는 있을것이지만, (synchronized 는 일반적으로 시스템 성능에 악영향을 미친다고 함).  
스트림 API 를 이용하면 쉽게 병렬성을 활용할 수 있다.


