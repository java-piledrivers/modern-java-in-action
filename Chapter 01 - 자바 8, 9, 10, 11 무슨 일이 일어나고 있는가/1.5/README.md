인터페이스

자바에서 인터페이스는 클래스들이 필수로 구현해야 하는 추상 자료형입니다.

=> 하나의 설계도면
=> ex) 동물 있다면 동물에 대한 설계도(움직이기, 먹기, 소리내기)


1.

public interface Animal {
    public static final String name = "동물";
    
    public abstract void move();
    public abstract void eat();
    public abstract void bark();
}


2.
public class Dog implements Animal{
    
    @Override
    public void move() {
        System.out.println("슥슥~~");
    }
    
    @Override
    public void bark() {
        System.out.println("멍멍!");
    }
}



하지만 sleep() 이라는 기능을 추가싶다면 


1.
public interface Animal {
    public static final String name = "동물";
    
    public abstract void move();
    public abstract void eat();
    public abstract void bark();
    public abstract  void sleep();
}

2.

public class Dog implements Animal{
    
    @Override
    public void move() {
        System.out.println("슥슥~~");
    }
    
    @Override
    public void bark() {
        System.out.println("멍멍!");
    }

    @Override
    public void sleep() {
        System.out.println("쿨쿨zzzz");
    }
}

해당 sleep 이라는 추상 메서드를 구현해주어야한다.


지금은 Animal 인터페이스를 구현한 Dog 클래스가 하나라 그렇지만 10~ 100개라면 해당 추상메서드를 전부다 구현해야 된다


이때를 위해 나온 메서드가 default 메서드이다.



1.

public interface Animal {
    public static final String name = "동물";
    
    public abstract void move();
    public abstract void eat();
    public abstract void bark();
	
    default void sleep(){
	System.out.println("쿨쿨zzzz");
	}
}


2.
public class Dog implements Animal{
    
    @Override
    public void move() {
        System.out.println("슥슥~~");
    }
    
    @Override
    public void bark() {
        System.out.println("멍멍!");
    }
}




3. public class Main {
	public static void main(String args[]){

	Dog dogTest = new Dog();
	dogTest.sleep();
	}
}



결과 : 쿨쿨zzzz



또한 인터페이스에서 구현된 default Method 는 구현클래스에서 재정의도 가능합니다.




=> 기존에 선언되고 구현된 메서드에 대하여 영향을 미치지 않기위해  => 하위호환성을 위해서 default Method 사용




그렇다면 추상클래스와 다를게 없지 않나요?

=>  클래스는 하나의 추상클래스를 상속 받지만 인터페이스는 여러개를 구현할수 있는 차이가 있습니다.



