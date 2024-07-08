# Factory 패턴
다양한 종류의 동물을 생성하는 예제와 함께 팩토리 패턴에 대해 알아보자.

## Example
> Animal 인터페이스
```java
public interface Animal {
    void speak();
}
```
> Animal 구현 클래스
```java
public class Cat implements Animal {
    @Override
    public void speak() {
        System.out.println("야옹~🐱");
    }
}
```
```java
public class Dog implements Animal {
    @Override
    public void speak() {
        System.out.println("멍멍!🐶");
    }
}
```
**Client**<br>
```java
Animal cat = new Cat();
Animal dog = new Dog();
```
**문제점**<br>
클라이언트에서 구현 클래스를 직접 의존한다. 구현클래스의 생성자가 변경되거나 전처리 코드가 변경되었을 때 클라이언트 코드를 수정해야 한다.

<br>

# SimpleFactory
객체의 생성만을 담당하는 별도의 Factory 클래스를 만들어보자.
```java
public class AnimalFactory {
    public Animal createAnimal(AnimalType animalType) {
        switch (animalType) {
            case CAT:
                return new Cat();
            case DOG:
                return new Dog();
            default:
                throw new IllegalArgumentException("Unknown animal type: " + animalType);
        }
    }
}
```
```java
public enum AnimalType {
    CAT, DOG
}
```
**Client**<br>
```java
AnimalFactory animalFactory = new AnimalFactory();
Animal cat = animalFactory.createAnimal(AnimalType.CAT);
Animal dog = animalFactory.createAnimal(AnimalType.DOG);
```
클라이언트에서 구현 클래스에 직접 의존하지 않게 되었다. 생성자가 변경되는 경우에도 AnimalFactory 클래스 내부만 수정하면 된다.

**문제점**<br>
새로운 Animal 구현 클래스가 추가되었을 때 createAnimal 기존 코드를 수정해야 한다.

<br>

# Factory Method
기존 코드에 영향을 주지 않고 확장 가능하도록(OCP) 팩토리 메서드 패턴을 적용해보자.

### Factory Method 패턴이란?
객체 생성 인터페이스를 정의하고, 구체적인 객체 생성 로직을 하위 클래스에 위임하는 디자인 패턴이다.
> AnimalFactory 인터페이스
```java
public interface AnimalFactory {
    Animal createAnimal();
}
```
> AnimalFactory 구현 클래스
```java
public class CatFactory implements AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Cat();
    }
}
```
```java
public class DogFactory implements AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Dog();
    }
}
```
**Client**<br>
```java
AnimalFactory catFactory = new CatFactory();
AnimalFactory dogFactory = new DogFactory();
Animal cat = catFactory.createAnimal();
Animal dog = dogFactory.createAnimal();
```
새로운 Animal 구현 클래스가 추가되어도 기존 코드를 수정하지 않고 확장할 수 있다.
예를 들어, Bird를 추가한다면 Bird 클래스를 정의하고 BirdFactory 클래스를 구현하면 된다.

---
**Reference**
- https://bcp0109.tistory.com/366
- https://donxu.tistory.com/entry/Factory-Method-Pattern%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C-%ED%8C%A8%ED%84%B4
