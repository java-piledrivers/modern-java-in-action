함수형 인터페이스 : 오직 하나의 추상 메서드를 가진다.

함수형 인터페이스의 추상메서드 : 함수형 디스크립터

함수형 인터페이스의 추상메서드는 람다표현식의 시그니처를 묘사(메서드 명과 파라미터의 순서, 타입, 개수)


다양한 함수형 디스크립터

1. Predicate

 - T라는 객체를 받아서 test라는 메서드로 Boolean 값을 리턴한다.

@FunctionalInterface
public interface Predicate<T>{
	boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p){
	List<T> results = new ArrayList<>();
	for(T t : list){
		if(p.test(t)){
			results.add(t);
		}
	}
	return results;
}
Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate)


2.Consumer
  
- T라는 객체를 받아서 accept라는 메서드로 void를 반환한다.
  
@FunctionalInterface
public interface Consumer<T> {
	void accept(T t);
}

public void <T> void forEach(List<T> list, Consumer<T> c) {
    for (T t : list) {
        c.accept(t);
    }
}

forEach(
  Arrays.asList(1,2,3,4,5),
  (Integer i) -> System.out.println(i); \
);


3.Function
  
-T라는 객체를 받아서 apply 라는 메서드로 제네릭 형식 R 객체를 반환한다.
  
@FunctionalInterface
public interface Function<T, R> {
	R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for (T t : list) {
        result.add(f.apply(t));
    }
    return result;
}

List<Integer> l = map(
    Arrays.asList("lambdas", "in", "action"),
    (String s) -> s.length(); 
);

박싱

- 기본형 -> 참조형

언박싱

-참조형 -> 기본형

오토박
  싱
-박싱와 언박싱이 자유롭게 수행

이러한 변환과정을 비용이 소모 

자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱을 피할수 있도록 함수형 인터페이스 제공
