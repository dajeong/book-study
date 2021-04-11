# 2장 객체 생성과 파괴

<br>
객체를 만들어야 할 때와 아닐 때, 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법, 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령 알아보기
<br>


## 1. 생성자 대신 정적 팩터리 메서드를 고려하라

다음과 같이 정적 팩터리 메서드(static factory method)로 인스턴스를 생성할 수 있다.

```java
// Boolean 클래스에 정의된 메서드
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean.FALSE;
}
```
<br>

정적 팩터리 메서드가 생성자보다 좋은 장점

1. 반환될 객체의 특성을 설명하는 이름을 가질 수 있다.<br>BigInteger.probablePrime(int bitLength, Random rnd) //소수 값의 BigInteger 반환<br>

2. 호출될 때마다 인스턴스를 새로 생성하지는 않을 수 있다. (재활용)<br>Boolean.valueOf의 return을 참고, 특히 생성 비용이 큰 객체가 자주 요구되는 경우 성능상 이점을 갖는다.<br>

3. 반환 타입의 하위 타입 객체를 반환할 수 있다. (유연성)<br>자바의 컬렉션 프레임워크가 제공하는 구현체 대부분을 java.util.Collections에서 정적 팩터리 메서드를 통해 얻는다. 사용하는 쪽에선 구현 클래스가아닌 인터페이스만으로 다루기 때문에 클라이언트는 인터페이스에 대한 지식만 있으면 된다.<br>

4. 입력 매개변수에 따라서 객체를 반환할 수 있다.<br>EnumSet 클래스는 오로지 정적 팩터리를 통해 객체를 반환받을 수 있다. 전달받는 원소 수에 따라 다른 하위 타입의 인스턴스를 반환하는데, 원소가 64개 이하면 RegularEnumSet을 65개 이상이면 JumboEnumSet의 인스턴스를 반환한다. 클라이언트는 EnumSet만 알면된다.<br>

5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.<br>이는 service provider framework 의 근간이되며 대표적인 예로 JDBC가 있다.<br>구현체의 동작을 정의하는 서비스 인터페이스 역할은 JDBC의 Connection,<br>제공자가 구현체를 등록할 때 사용하는 제공자 등록 API 역할은 JDBC의 DriverManager.registerDriver,<br>클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API 역할은 JDBC의 DriverManager.getConnection,<br>서비스 제공자 인터페이스 역할은 JDBC의 Driver 이다.<br>

정적 팩터리 메서드의 단점

1. 상속을 하려면 public 이나 protected 생성자가 필요하므로 정적 팩터리 메서드만 제공하는 경우 하위 클래스를 만들 수 없다.<br>

2. 자바독에서 볼 수 없으므로 프로그래머가 찾기 어렵다. API 문서를 잘 작성하고 메서드 이름도 널리 알려진 규약을 따라 짓자.
   - Date d = Date.of(instant);
   - Set\<Rank\> faceCards = EnumSet.of(JACK, QUEEN, KING);
   - BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
   - StackWalker luke = StackWalker.getInstance(options);
   - Object new Array = Arrays.newInstance(classObject, arrayLen);
   - FileStore fs = Files.getFileStore(path);
   - BufferedReader br = Files.newBufferedReader(path);
   - List\<Complaint\> litany = Collections.list(lagacyLitany);

정적 팩터리 메서드 or public 생성자? 각자 쓰임새에 따라 장단점 이해하고 사용하기. 정적 팩터리를 사용하는게 유리한 경우가 많음. [lombok 사용시 참고](https://projectlombok.org/features/constructor)<br><br>

## 2. 생성자에 매개변수가 많다면 빌더를 고려하라

<br>
선택적 매개변수가 많을 때 정적 팩터리나 생성자나 적절히 대응이 어렵다.
이를 대응하기위한 점층적 생성자 패턴(telescoping constructor pattern)도 마찬가지로 매개변수 개수가 많아지면 코드 작성과 읽기가 어렵고 자바 빈즈 패턴에서는 객체 생성을 위해 메서드 여러개를 호출해야하고 불변 객체로 만들기 어렵다.<br><br>

세 번째 대안 빌더 패턴, 예제 코드와 주석으로 설명 요약

```java
/**
* NutritionFacts 클래스는 불변 객체로 생성된다.
*/
public class NutritionFacts {
  private final int servingSize;
  private final int servings;
  private final int colories;
  private final int fat;

  public static class Builder {
    // 필수
    private final int servingSize;
    private final int servings;
    // 선택
    private final int calories = 0;
    private final int fat = 0;
    
    public Builder(int servingSize, int servings) {
      this.servingSize = servingSize;
      this.sergings = servings;
    }

    public Builder calories(int val) { calories = val; return this; } // 빌더 자신을 반환하면서 메서드 체이닝 가능

    public Builder fat(int val) { fat = val; return this; }

    public NutritionFacts build() { return new NutritionFacts(this); }
  }

  private NutritionFacts(Builder builder) {
    servgingSize = builder.servingSize;
    servgingSize = builder.servings;
    servgingSize = builder.calories;
    servgingSize = builder.fat;
  }
}
```
```java
// 빌더를 사용하는 클라이언트 코드
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).build(); // 선택 필드는 제외 가능
```
<br>
빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다. 피자의 다양한 종류를 표현하는 계층구조 예제를 보자.

```java
public abstract class Pizza {
  public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE };
  final Set<Topping> toppings;

  // 재귀적 타입 한정(아이템30)을 이용하는 제네릭 타입
  abstract static class Builder<T extends Builder<T>> {
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
    public T addTopping(Topping topping) {
      toppings.add(Objects.requireNonNull(topping));
      return self();
    }
    abstract Pizza build();
    protected abstract T self();  // 하위 클래스에서 형변환 없이 메서드 체이닝 지원 (simulated self-type)
  }
  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone();  // 아이템 50 참조
  }
}
```

```java
public class NyPizza extends Pizza {
  public enum Size { SMALL, MEDIUM, LARGE, FAMILY }
  private final Size size;

  public static class Builder extends Pizza.Builder<Builder> {
    private final Size size;
    public Builder(Size size) {
      this.size = Objects.requreNonNull(size);
    }
    @Override public NyPizza build() { 
      return new NyPizza(this); // 상위 클래스 정의 메서드의 반환 타입이아닌 하위 타입을 반환 - 공변 반환 타이핑(covariant return typing)
    }
    @Override protected Builder self() { return this; }
  }
  private NyPizza(Builder builder) {
    super(builder);
    size = builder.size;
  }
}
```

```java
NyPizza pizza = new NyPizza.builder(FAMILY).addTopping(SAUSAGE).addTopping(ONION).builder();
// addTopping 메서드는 호출할 때 마다 하나의 필드로 모은다. 기본 생성자에서는 못해
```

객체를 만들기 앞서 빌더부터 만드는 것은 성능이 아주 민감할 때 문제가 될 수 있다.  
<br>

## 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라  

싱글턴을 만드는 방식은 보통 둘 중 하나이고 모두 생성자는 private으로 감춰두고 인스턴스에 접근할 수 있는 유일한 public static 멤버 하나를 만든다.  

싱글턴을 만드는 방법 첫 번째, public static 멤버가 final 필드인 방식.
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {...} // Elvis.INSTANCE 초기화할 때 딱 한번만 호출된다. 리플렉션에 의한 생성을 방어하려면 두 번째 호출 시 예외를 던지자.
}
```  

싱글턴을 만드는 방법 두 번째, 정적 팩터리 메서드를 public static 멤버로 제공.  
```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();  // private 필드
  private Elvis() {...}
  public static Elvis getInstance() { return INSTANCE; }  // 항상 같은 객체 반환
}
```  
두 번째 방식의 장점 
- API 변경 업이 싱글턴이 아니게 변경할 수 있다.
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
- 정적 팩터리의 메서드 참조를 Supplier로 사용할 수 있다. (ex. public static Supplier(Elvis) getInstance() {...})

위 두 방식으의 싱글턴 클래스를 직렬화 할 때 Serialize 구현 이외에도 모든 필드를 transient로 선언하고 readResolve 메서드를 제공해야한다. 이렇게 하지 않으면 역직렬화할 때 마다 새로운 인스턴스가 만들어진다.  

싱글턴을 만드는 방법 세 번째, 원소가 하나인 열거 타입을 선언.
```java
public enum Elvis {
  INSTANCE;
}
```
간결, 직렬화 쉬움, 리플렉션 공격 안먹힘. Enum 외 클래스 상속해야하는 경우 제외하고 대부분 상황에서 이 방식이 가장 Best.

## 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

## 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

## 6. 불필요한 객체 생성을 피하라

## 7. 다 쓴 객체 참조를 해제하라

## 8. finalizer와 cleaner 사용을 피하라

## 9. try-finally보다는 try-with-resources를 사용하라