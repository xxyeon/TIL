## SOLID 원칙

### OCP

기존 코드를 변경하지 않으면서, 기능을 추가할 수 있도록 설계되어야 한다는 원칙이다.
확장이란 새로운 기능이 추가됨을 의미하는데, 기능 추가를 위해 확장을 통해 구현하여, 확장에 따른 클래스 수정은 최소화하도록 프로그램을 작성해야한는 원칙
쉽게 생각해서 추상화를 의미한다고 생각하면 된다.
전략패턴이 그 예이다.

```java
// 추상화
abstract class Animal {
    abstract void speak();
}

class Cat extends Animal { // 상속
    void speak() {
        System.out.println("냐옹");
    }
}

class Dog extends Animal { // 상속
    void speak() {
        System.out.println("멍멍");
    }
}

class HelloAnimal { //기능이 추가되어도 HelloAnimal 의 코드가 수정되지 않는다.
    void hello(Animal animal) {
        animal.speak();
    }
}

public class Main {
    public static void main(String[] args) {
        HelloAnimal hello = new HelloAnimal();

        Animal cat = new Cat();
        Animal dog = new Dog();

        hello.hello(cat); // 냐옹
        hello.hello(dog); // 멍멍
    }
}
```

### OCP 원칙을 따른 JDBC

> 데이터베이스 인터페이스인 JDBC가 OCP 원칙을 잘 따른 예시
> 자바 애플리케이션에서 사용하고 있는 데이터베이스를 MySQL에서 Oracle로 바꾸고 싶다면, 복잡한 하드코딩 없이 connection객체만 변경하면 된다.
