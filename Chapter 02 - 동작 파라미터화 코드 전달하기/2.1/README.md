# 동적 파라미터화 코드 전달하기

### 2.1 변화하는 요구사항에 대응

**우리가 어떤 상황에서 일을 하든 소비자 요구사항은 항상 바뀐다.**

→ 1. 개발자는 새로 추가한 기능은 쉽게 구현할 수 있어야 하며 장기적인 관점에서 유지보수가 쉬워야 한다 

→ 2. 엔지어링적인 비용이 가장 최소하 될 수 있으면 더 좋다.

1, 2를 충족시킬 수 있는 해결방법이 `동작 파라미터화`이다.

`동작 파라미터화`란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다.

예를 들어 나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있다. 즉, 코드 블록에 따라 메서드의 동작이 파라미터화 된다. 

변화에 대응하는 코드를 구현하는 것은 어려운 일이다. 그 이유는 예시를 통해서 알아보자.

**예시 : 기존 농장 재고목록 애플리케이션에서 사과에 대한 필터링** 

**첫 번째 상황) 녹색 사과만 필터링**

```java
enum color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            **if (GREEN.eqauls(apple.getColor())) {**
                result.add(apple);
            }
        }
        return result;
    }

```

**두 번째 상황) 빨간 사과만 필터링**

단순한 방법 : filterReadApples 메서드를 만들고 RED에 대해서 필터링을 할 것이다.

하지만, 농부가 옅은 녹색, 어두운 빨간색, 노란색 등 요구사항이 변경할 때마다 메서드를 반복해서 복사해야 하기 때문에 적절하지는 않다.

이런 상황에서는 다음과 같은 좋은 규칙이 있다.

**`거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화 한다.`**

**파라미터화)**

어떻게 해야 filterGreenApples의 코드를 반복 사용하지 않고 다양한 색깔의 사과를 구현할 수 있을까?

이것 또한 단순하게 생각해보면 파라미터에 색깔 추가하면 된다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (color.eqauls(apple.getColor())) {
                result.add(apple);
            }
        }
        return result;
    }

```

이제 농부의 요구사항은 다음과 같은 코드를 통해 해결할 수 있다.

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

그런데, 농부가 갑자기 색 이외에도 가벼운 사과와 무거운 사과로 구분할 수 있으면 좋겠습니다. 보통 무게가 150그램 이상인 사과가 무거운 사과입니다. 라고 요구한다.

여기서 눈치챘겠지만 소비자의 요구사항은 늘 변한다는 것을 알 수 있다. 

그러면 이러한 요구사항 또한 weight 파라미터를 추가하면 충분히 해결할 수 있을 것이다.

하지만 목록을 검색하고, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복될 것이다. 이는 소프트웨어 공학의 DRY(don’t repeat yourself, 같은 것을 반복하지 말것) 원칙을 어기는 것이다.

탐색 과정을 고쳐서 성능을 개선할면 무슨 일이 일어나야 할까? 한 줄이 아니라 메서드 전체 구현을 고쳐야 한다. 즉, 엔지니어링적으로 비싼 대가를 치러야 한다.

색과 무게를 filter라는 메서드로 합치는 방법도 있다. 그러면 어떤 기준으로 사과를 필터링할지 구분하는 또 다른 방법이 필요하다. 따라서 색이나 무게 중 어떤 것을 기준으로 필터링할지 가리키는 플래그를 추가할 수 있다(하지만 실전에서는 절대 이 방법을 사용하지 말아야 한다. 뒤에서 설명)

### 가능한 모든 속성으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (flag && color.eqauls(apple.getColor()) ||) {
                (!flag && apple.getWeight() > weight))
                result.add(apple);
            }
        }
        return result;
    }
```

이제 위 메서드를 사용할 수 있다

```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```

과연 이게 좋은 코드일까?

대체 true와 false는 뭘 의마하는 걸까? ( 이것은 final을 통해 이름을 정의하면 어느정도 해결은 된다)

게다가 앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수도 없다. 예를 들어 사과의 크기, 모양, 출하지 등으로 사과를 필터링 하고 싶다면 어떻게 될까? 

결국 여러 중복된 필터 메섣르르 만들거나 아니면 모든 것을 처리하는 거대한 하나의 필터 메서드를 구현해야 한다.

문제가 잘 정의되어 있는 사황에서는 위처럼 해도 되기는 한다. 하지만 filterApples에 어떤 기준으로 사과를 필터링할 것인지 효과적으로 전달할 수 있다면 더 좋을 것이다. 이는 `동적 파라미터화`가 해결 해준다.
