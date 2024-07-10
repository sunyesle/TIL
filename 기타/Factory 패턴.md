# Factory 패턴

객체의 생성 절차가 복잡하거나 다양한 조건에 따라 객체를 생성해야 할 때, 팩토리 메서드를 사용하면 생성 로직을 캡슐화할 수 있다. (관심사의 분리)

객체를 생성하는 방법이 한 가지 이상일 경우, 생성자는 클래스 이름과 동일해야 하므로 그 의미가 모호해질 수 있다. 그러나 팩토리 메서드를 사용하면 메서드 이름을 통해 생성되는 객체의 용도나 생성 방식을 명확히 전달할 수 있다.

## Example
다양한 종류의 동물을 생성하는 예제와 함께 팩토리 패턴에 대해 알아보자.
```java
// Animal 인터페이스
public interface Animal {
    void speak();
}

public class Cat implements Animal {
    @Override
    public void speak() {
        System.out.println("야옹~🐱");
    }
}

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

![factoryMethod_structure](https://github.com/sunyesle/TIL/assets/45172865/85c3cc3e-0408-4b90-9c6e-0fa96a0e8b38)

```java
// AnimalFactory 인터페이스
public interface AnimalFactory {
    Animal createAnimal();
}

public class CatFactory implements AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Cat();
    }
}

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

<br>

# Abstract Factory

### Abstract Factory 패턴이란?
관련된 객체들을 생성하기 위한 인터페이스를 정의하고, 객체들을 생성하는 책임을 하위 클래스에 위임하는 디자인 패턴이다.

![abstractFactory_structure](https://github.com/sunyesle/TIL/assets/45172865/76a77062-cfa3-46ad-aa88-21db7b52dcdd)

## Example
GUI 라이브러리에서 운영체제에 따라 다른 UI 컴포넌트를 생성하는 예제와 함께 추상 팩토리 패턴에 대해 알아보자.
> Button
```java
public interface Button {
    void paint();
}

public class WindowsButton implements Button {
    @Override
    public void paint() {
        System.out.println("Windows 버튼 생성");
    }
}

public class MacOSButton implements Button{
    @Override
    public void paint() {
        System.out.println("MacOS 버튼 생성");
    }
}
```
> CheckBox
```java
public interface CheckBox {
    void paint();
}

public class WindowsCheckBox implements CheckBox{
    @Override
    public void paint() {
        System.out.println("Windows 체크박스 생성");
    }
}

public class MacOSCheckBox implements CheckBox{
    @Override
    public void paint() {
        System.out.println("MacOS 체크박스 생성");
    }
}
```
> GUIFactory
```java
// 추상 팩토리
public interface GUIFactory {
    Button createButton();
    CheckBox createCheckBox();
}

public class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    @Override
    public CheckBox createCheckBox() {
        return new WindowsCheckBox();
    }
}

public class MacOSFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacOSButton();
    }

    @Override
    public CheckBox createCheckBox() {
        return new MacOSCheckBox();
    }
}
```
**Client**<br>
```java
public class AbstractFactoryApp {
    public static void main(String[] args) {
        Application app = new Application(new WindowsFactory());
        app.paint();
    }

    public class Application {
        private Button button;
        private Checkbox checkbox;

        public Application(GUIFactory factory) {
            button = factory.createButton();
            checkbox = factory.createCheckbox();
        }

        public void paint() {
            button.paint();
            checkbox.paint();
        }
    }
}
```
관련된 객체들의 생성 로직을 하나의 팩토리에 모아둘 수 있다.

---
**Reference**
- https://bcp0109.tistory.com/366
- https://donxu.tistory.com/entry/Factory-Method-Pattern%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C-%ED%8C%A8%ED%84%B4
- https://refactoring.guru/ko/design-patterns/factory-comparison
- https://oobwrite.com/entry/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4-%EC%B6%94%EC%83%81-%ED%8C%A9%ED%86%A0%EB%A6%AC-%ED%8C%A8%ED%84%B4-%EA%B0%9D%EC%B2%B4-%EC%83%9D%EC%84%B1%EC%9D%98-%EC%9C%A0%EC%97%B0%EC%84%B1%EA%B3%BC-%ED%99%95%EC%9E%A5%EC%84%B1-%EA%B7%B9%EB%8C%80%ED%99%94
- https://jake-seo-dev.tistory.com/366
