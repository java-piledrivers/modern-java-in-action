# 11.1 값이 없는 상황을 어떻게 처리할까?
- 아래 코드에선 차를 소유하지 않은 사람, 차는 있지만 보험 미가입인 경우 등 NullPointerException이 발생할 가능성이 있다.
```java
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```
<br>

## 11.1.1 보수적인 자세로 NullPointerException 줄이기
- 예기치 않은 NullPointerException을 피하려면? <br> 대부분의 프로그래머는 다양한 null 확인 코드를 추가해서 null 예외 문제를 해결하려 할 것이다.
```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```
- 위 코드는 변수를 참조할때마다 null을 확인한다. 중첩 if가 추가되면서 코드 들여쓰기 수준이 증가한다.<br>
- 이런 반복 패턴을 '깊은 의심'이라 부르며, 코드의 구조가 엉망이 되고 가독성도 떨어진다.<br>

```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        return "Unknown";
    }

    Car car = person.getCar();
    if (car != null) {
        return "Unknown";
    }

    Insurance insurance = car.getInsurance();
    if (insurance != null) {
        return "Unknown";
    }

    return insurance.getName();
}
```
- 위 코드는 조금 다른 방법으로 중첩 if 블록을 없앴다.<br>
- 메서드에 네 개의 출구가 생겨 코드의 유지보수가 어려워진다.<br>
  null일 때 반환되는 기본값 'Unknown'이라는 같은 문자열이 세 곳에서 반복되어 오타 등의 실수가 생길 수 있다.
  
<br>

## 11.1.2 null 때문에 발생하는 문제
자바에서 null 레퍼런스를 사용하면서 발생할 수 있는 이론적, 실용적 문제
- 에러의 근원이다 : NullPointerException은 자바에서 가장 흔히 발생하는 에러다.
- 코드를 어지럽힌다 : 중첩된 null 확인 코드로 인해 가독성이 떨어진다.
- 아무 의미가 없다 : 아무 의미도 표현하지 않는 null은, 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.
- 자바 철학에 위배된다 : 자바는 개발자로부터 모든 포인터를 숨겼으나, null 포인터만은 예외가 되었다.
- 형식 시스템에 구멍을 만든다 : 모든 참조 형식에 null 할당이 가능하여, 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 어떤 의미로 사용되었는지 알 수 없다.

```text
정적 형식 언어?
자료형이 컴파일 시에 결정되는 언어.
프로그래머가 변수에 들어갈 값의 형태에 따라 직접 변수의 타입을 명시해야 한다.
컴파일 시 자료형에 맞지 않는 값이 들어있으면 컴파일 에러가 발생한다.
```

<br>

## 11.1.3 다른 언어는 null 대신 무얼 사용하나?
- 그루비 : 안전 내비게이션 연산자 (?.)
호출 체인 중 null인 참조가 있을 시 결과로 null이 반환된다.
```java
def carInsuranceName = person?.car?.insurance?.name
```
- 하스켈 : 선택형값(optional value)을 저장할 수 있는 Maybe 형식<br>
주어진 형식의 값을 갖거나, 아무 값도 갖지 않는다. -> null 참조 사라짐.
- 스칼라 : Option[T]<br>
T 형식의 값을 갖거나 아무 값도 갖지 않는다.<br><br>

자바 8은 선택형값 개념의 영향을 받아서 java.util.Optional라는 새로운 클래스를 제공한다.<br>
이 장에서는 java.util.Optional<T>를 이용해 값이 없는 상황을 모델링하는 방법을 설명한다.<br>
  

