# 디폴트 메서드

자바에서 인터페이스를 사용하면 해당 인터페이스를 구현하는 클래스는 인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 아니면 슈퍼클래스의 구현을 상속받아야 한다.  
만약 내가 라이브러리 설계자가 되어서 인터페이스에 새로운 메서드를 추가하는 등 인터페이스를 변경하고자 한다면 해당 인터페이스를 구현했던 모든 클래스의 구현도 고쳐야 하는 문제가 발생한다.

이러한 문제를 해결하고자 자바 8에서는 인터페이스를 정의하는 두 가지 방법을 제공한다.  

1. 정적 메서드(static method)
2. 디폴트 메서드(default method)

자바 8에서는 메서드 구현을 포함하는 인터페이스를 정의할 수 있다. 때문에 **기존 인터페이스를 구현하는 클래스는 자동으로 인터페이스에 추가된 새로운 메서드의 디폴트 메서드를 상속받게 된다.** 다시 말해 기존의 코드 구현을 바꾸지 않으면서 인터페이스를 바꿀 수 있는 것이다.  

아래는 List 인터페이스의 sort 메서드 구현 코드이다. 

```java
default void sort(Comparator<? super E> c) {
	Collections.sort(this, c);
}
```

sort 메서드는 Collections.sort 메서드를 호출한다. 이 새로운 디폴트 메서드 덕분에 리스트에 직접 sort를 호출할 수 있게 되었다.

반환 형식 void 앞에 default 라는 키워드가 붙었다. 여기서 default는 해당 메서드가 디폴트 메서드임을 명시한다. 접근제어자에서 사용하는 default 와 같은 키워드지만, 접근제어자는 아무것도 명시하지 않은 접근제어자를 default라고 하고, 인터페이스의 default method는 default 키워드를 명시해야 한다.  


```java
List<Integer> numbers = Arrys.asList(3, 5, 1, 2, 6);
numbers.sort(Comparator.naturalOrder());  // sort는 List 인터페이스의 디폴트 메서드
```

위 코드에서 `Comparator.naturalOrder` 이라는 새로운 메서드가 등장했다. 이 메서드는 Comparator 인터페이스에 새로 추가된 정적메서드이다. 자연 순서(표준 알파벳 순서)로 요소를 정렬할 수 있도록 Comparator 객체를 반환한다. 
  
    


디폴트 메서드의 장점은 **인터페이스의 기본 구현을 그대로 상속하면서 인터페이스에 자유롭게 새로운 메서드를 추가**할 수 있다는 점이다.


## 13.1 변화하는 API

상황을 예로 들어보자.

현재 우리는 자바 드로잉 라이브러리 설계자가 되었다. 이 라이브러리에는 모양의 크기를 조절하기 위해 필요한 getter, setter 메서드를 정의하는 Resizable 인터페이스가 있다. 그리고 직사각형이나 정사각형처럼 Resizable을 구현하는 클래스도 제공한다. 라이브러리가 인기를 끌면서 일부 사용자가 직접 Resizable 인터페이스를 구현하는 Ellipse 라는 클래스를 구현하기도 했다.  

API를 릴리스한 지 몇개월이 지나고 Resizable에 크기 조절 인수로 모양의 크기를 조절할 수 있는 setRelativeSize 메서드가 있으면 좋을 것 같아서 해당 메서드를 추가하고 Square와 Recatangle 구현도 고쳤다. 여기서 끝인가? 아니다.  
라이브러리 사용자가 만든 클래스를 우리가 어떻게 할 수는 없다. 

코드로 구체적으로 살펴보자.


### 13.1.1 API 버전 1


Resizable 인터페이스

```java

public interface Resizable extends Drawable {
	int getWidth;
    int getHeight;
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
}
```


사용자 구현 클래스 Ellipse

```java
public class Ellipse implements Resizable {
	...
}
```


```java
public class Game {
	public static void main(String[] args) {
    List<Resizable> resizableShapes =
    	Arrays.asList(new Square(), new Rectagle(), new Ellipse());
    Utils.paint(resizableShapes);
    }
}

public class Utils {
	public static void paint(List<Resizable> l) {
    	l.forEach(r -> {
        	r.setAbsoluteSize(42,42);  // 각 모양에 setAbsoluteSize 호출
            r.draw();
		});
	}
}
```

  
  
### 13.1.2 API 버전 2

사용자의 요구에 따라 setRelativeSize 메서드를 추가하였다.

Resizable 인터페이스 버전 2

```java

public interface Resizable extends Drawable {
	int getWidth;
    int getHeight;
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    void setRelativeSize(int wFactor, int hFactor); // 새로 추가된 친구
}
```

**사용자가 겪는 문제**

Resizable을 고치면 어떤 문제점이 생기는지 살펴보자.

첫번째로는 Resizable 을 구현한 모든 클래스는 setRelativeSize 메서드를 구현해야 한다는 문제가 발생한다. 그렇지만 라이브러리 사용자가 직접 구현한 Ellipse 클래스 setRelativeSize 메서드를 구현하지 않는다. 
**인터페이스에 새로운 메서드를 추가하면 바이너리 호환성은 유지된다.** 바이너리 호환성이란, 
새로 추가된 메서드를 호출하지만 않으면 새로운 메서드 구현이 없이도 기존 클래스 파일 구현이 잘 동작한다는 의미이다. 
그렇지만 누군가 Utils.paint에서 setRelativeSize 를 사용하도록 코드를 수정한다면 문제가 되고 이때는 런타임 에러 `java.lang.AbstractMethodError` 가 발생할 것이다. 


두번째로는 사용자가 Ellipse를 포함하는 전체 애플리케이션을 재빌드할 때 컴파일 에러가 발생한다.  

디폴트 메서드로 이 모든 문제를 해결할 수 있다. 디폴트 메서드를 이용해 API 를 바꾸면 새롭게 바뀐 인터페이스에서 자동으로 기본 구현을 제공하므로 기존 코드를 고치지 않아도 된다. 
