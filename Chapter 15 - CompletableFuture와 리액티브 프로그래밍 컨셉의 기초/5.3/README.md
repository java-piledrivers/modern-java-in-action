## 5.3 박스와 채널 모델

## 박스와 채널 모델은 동시성을 설계하고 계념화하기 위한 모델이다.

## 박스와 채널 모델을 이용하면 생각과 코드를 구조화할 수 있으며, 시스템 구현의 추상화 수준을 높일 수 있다.

## 박스로 원하는 연산을 표현하면 계산을 손으로 코딩한 결과보다 더 효율적일 것이다.

## 또한 병렬성을 직접 프로그래밍하는 관점을 콤비네이터를 이용해 내부적으로 작업을 처리하는 관점으로 바꿔준다.



![image](https://github.com/java-piledrivers/modern-java-in-action/assets/93499421/6f6f39a7-7eda-4c6c-b8a8-d36a90adb579)







```java
int t = p(x);
System.out.println(r(q1(t), q2(t));
// 위 방식은 q1, q2를 차례로 호출하여 하드웨어 병렬성 활용과는 거리가 멀다.


int t = p(x);
Future<integer> a1 = executorService.submit(() -> q1(t));
Future<integer> a2 = executorService.submit(() -> q2(t));
System.out.println(r(a1.get(), a2.get());
    ```
    

## 많은 태스크가 get() 메서드를 호출해서 Future가 끝나기를 기다리게 되면 하드웨어의 병렬성을 제대로 활용하지 못하거나 데드락에 걸릴 수도 있다.
