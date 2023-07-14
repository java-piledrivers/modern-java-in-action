# 19.2 영속 자료구조

**목표: *함수형 프로그램에서 사용하는 자료구조를 살펴보자***

## 파괴적인 갱신과 함수형

```java
// 590p 예제 코드
  
public class TrainJourney {  
    public int price;  
    public TrainJourney onward;  
    public TrainJourney(int p, TrainJourney t) {  
        price = p;  
        onward = t;  
    }  
  
    // 파괴적 갱신: 기존 객체를 수정  
    static TrainJourney link(TrainJourney a, TrainJourney b) {  
        if (a == null) {  
            return b;  
        }  
        TrainJourney t = a;  
        while (t.onward != null) {  
            t = t.onward;  
        }  
        t.onward = b;  
        return a;  
    }  
  
    // 함수형 프로그래밍: 새로운 객체를 만들어서 반환  
    static TrainJourney append(TrainJourney a, TrainJourney b) {  
        return a == null ? b : new TrainJourney(a.price, append(a.onward, b));  
    }  
}
```

**link() 메서드**
- 방식: 파괴적 갱신 (Destructive Updates)
- 작동 원리: 첫 번째 TrainJourney 객체 `a`의 onward 리스트의 마지막에 TrainJourney 객체 `b`를 직접 연결.
- 특징: 이 과정에서 원래의 TrainJourney 객체 `a` 자체가 변경됨.

**append() 메서드**
- 방식: 함수형
- 작동 원리: 새로운 TrainJourney 객체를 생성하고, 그 객체의 onward를 재귀적으로 다음 TrainJourney로 설정
- 특징: 원래의 TrainJourney 객체 `a`는 변경되지 않음

## 트리를 사용한 다른 예제

```java
public class Tree {  
    public String key;  
    public int val;  
    public Tree left, right;  
    public Tree(String k, int v, Tree l, Tree r) {  
        key = k;  
        val = v;  
        left = l;  
        right = r;  
    }  
}  
  
public class TreeProcessor {  
    public static int lookup(String k, int defaultval, Tree t) {  
        if (t == null) {  
            return defaultval;  
        }  
        if (k.equals(t.key)) {  
            return t.val;  
        }  
        return lookup(k, defaultval, k.compareTo(t.key) < 0 ? t.left : t.right);  
    }  
  
    public static Tree update(String k, int newval, Tree t) {  
        if (t == null) {  
            // 새로운 노드를 만들어서 반환  
            return new Tree(k, newval, null, null);  
        } else if (k.equals(t.key)) {  
            t.val = newval;  
        } else if (k.compareTo(t.key) < 0) {  
            t.left = update(k, newval, t.left);  
        } else {  
            t.right = update(k, newval, t.right);  
        }  
        return t;  
    }  
}
```

위 코드는 Tree 자체를 변경시킨다. (파괴적 갱신으로 보임)

```java
public static Tree fupdate(String k, int newval, Tree t) {  
    return (t == null) ?  
            new Tree(k, newval, null, null) :  
            k.equals(t.key) ?  
                    new Tree(k, newval, t.left, t.right) :  
                    k.compareTo(t.key) < 0 ?  
                            new Tree(t.key, t.val, fupdate(k, newval, t.left), t.right) :  
                            new Tree(t.key, t.val, t.left, fupdate(k, newval, t.right));  
}
```

위 코드는 순수한 함수형이다. 새로운 Tree 를 만든다.

Q) 보통 update 라고 하면, 기존 객체나 데이터를 수정하거나 갱신하는 작업을 가리키는 경우가 많지 않나?  새로 객체를 리턴한다면 update가 아닌것 같은데..
