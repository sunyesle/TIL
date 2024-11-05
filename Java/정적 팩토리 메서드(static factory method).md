# 정적 팩토리 메서드(static factory method)
> *Effective Java - ITEM 1*<br>
> 생성자 대신 정적 팩토리 메서드를 고려하라

## 요약
객체생성을 캡슐화 하는기법이다.

객체를 생성하는 메서드를 만들고 static으로 선언한다.

### 장점
- 이름을 가질 수 있어 생성자보다 가독성이 좋다.
- 호출할 때마다 새로운 객체를 생성하지 않아도 된다.
- 하위 자료형 객체를 반환할 수 있다.
### 단점
- 정적 팩토리 메서드만 있는 클래스라면, 생성자가 없으므로 하위 클래스를 만들지 못한다.
- 정적 팩토리 메서드는 다른 정적 메서드와 잘 구분되지 않는다.

## 장점
### 이름을 가질 수 있어 생성자보다 가독성이 좋다
```java
public class Character {
    private int intelligence;
    private int strength;
    private int hitPoint;
    private int magicPoint;

    private Character(int intelligence, int strength, int hitPoint, int magicPoint) {
        this.intelligence = intelligence;
        this.strength = strength;
        this.hitPoint = hitPoint;
        this.magicPoint = magicPoint;
    }

    public static Character createWarrior() {
        return new Character(5, 15, 20, 3);
    }

    public static Character createMage() {
        return new Character(15, 5, 10, 15);
    }
```

생성자를 사용해 전사나 마법사를 생성하는 경우, 변수명이 없었다면 캐릭터의 직업을 알아보기 어려웠을 것이다.
```java
Character warrior = new Character(5, 15, 20, 3);
Character mage = new Character(15, 5, 10, 15);
```

하지만 정적 팩토리 메서드를 사용한다면 좀 더 읽기 쉬운 코드가 된다.
```java
Character warrior = Character.createWarrior();
Character mage = Character.createMage();
```

### 호출할 때마다 새로운 객체를 생성하지 않아도 된다
immutable 객체를 캐시해두고 캐싱된 객체를 반환한다.

자바의 BigInteger의 valueOf는 그 예 중 하나이다.
```java
public static final BigInteger ZERO = new BigInteger(new int[0], 0);

private final static int MAX_CONSTANT = 16;
private static BigInteger posConst[] = new BigInteger[MAX_CONSTANT+1];
private static BigInteger negConst[] = new BigInteger[MAX_CONSTANT+1];

static {
    /* posConst에 1 ~ 16까지의 BigInteger 값을 담는다. */
    /* negConst에 -1 ~ -16까지의 BigInteger 값을 담는다. */
}

public static BigInteger valueOf(long val) {
    // 미리 만들어둔 객체를 리턴한다
    if (val == 0)
        return ZERO;
    if (val > 0 && val <= MAX_CONSTANT)
        return posConst[(int) val];
    else if (val < 0 && val >= -MAX_CONSTANT)
        return negConst[(int) -val];

    // 새로운 객체를 만들어 리턴한다
    return new BigInteger(val);
}
```

### 하위 자료형 객체를 반환할 수 있다
```java
public class OrderUtil {
    public static Discount createDiscountItem(String discountCode) {
        if (!isValidCode(discountCode)) {
            throw new Exception("잘못된 할인 코드");
        }

        if (isUsableCoupon(discountCode)) {
            return new Coupon(1000);
        } else if (isUsablePoint(discountCode)) {
            return new Point(500);
        }
        throw new Exception("이미 사용한 코드");
    }
}

class Discount { }
class Coupon extends Discount { }
class Point extends Discount { }
```

## 단점
### 정적 팩토리 메서드는 다른 정적 메서드와 잘 구분되지 않는다
널리 사용되는 메서드 명명법을 지켜 단점을 극복할 수 있다.

### 대표적인 메서드 명명 방식
- `from`
  - 매개변수를 하나 받아 해당 타입의 인스턴스를 반환하는 메서드에 주로 사용한다.
  - Date date = Date.from(instance);
- `of`
  - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드에 주로 사용한다. 
  - List list = List.of(1, 2, 3);
- `valueOf`
  - Integer i = Integer.valueOf(10)
- `instance` of `getInstance`
  - 해당 요청에 맞는 인스턴스를 반환하는 메서드에서 주로 사용한다.
- `create` or `newInstance`
  - instance와 비슷한 의미지만 매번 새로운 인스턴스 생성을 보장할 때 주로 사용한다.
  - Object newArray = Array.newInstance(Integer.class, 10);
- `getType`
  - getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 사용한다.
  - *Type*은 팩토리가 반환할 객체의 타입
  - FileStore fileStore = Files.getFileStore(path);
- `newType`
  - newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 사용한다.
  - BufferedReader bufferedReader = Files.newBufferedReader(path);
- `type`
  - getType, newType을 간결하게 사용할 때  

---
**Reference**<br>
- https://johngrib.github.io/wiki/pattern/static-factory-method
- https://sun-22.tistory.com/84
