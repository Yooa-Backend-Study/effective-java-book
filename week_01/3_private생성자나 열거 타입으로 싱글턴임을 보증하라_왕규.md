# Singleton
싱글턴(singleton) 디자인 패턴이라는 말을 많이 들어보셨을 겁니다. <br>
싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말합니다. <br>

>#### 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.
다음과 같이 코드로 알아보겠습니다

```java
public class SingletonTest {

    @Test
    public void testSingleton() {
        // 이 테스트는 싱글톤 객체를 테스트하려고 함

        // Given
        Singleton singleton = Singleton.getInstance();
        singleton.setValue(42);

        // When
        int result = singleton.getValue();

        // Then
        assertEquals(42, result);
    }
}
```
테스트가 이 클래스의 실제 싱글톤 인스턴스에 의존하게 되며, 상태 변경이 **다른 테스트**에 영향을 줄 수 있습니다. <br>
싱글톤 객체가 다른 테스트에 의해 변경되면 테스트 결과가 예측 불가능해질 수 있으며, 디버깅과 유지 관리가 어려워집니다. <br>
이러한 이유로 테스트 가능한 코드를 작성하려면 싱글톤 패턴을 사용하는 것 대신 의존성 주입과 같은 디자인 패턴을 고려하고, 테스트용 객체를 주입하면서 클래스를 테스트하기가 더 쉬워집니다. <br>


## 싱글턴의 장단점
#### 장점
- 싱글톤 패턴을 사용함으로써 얻을 수 있는 이점 중 하나는 메모리 낭비를 방지할 수 있다 <br>
DBCP(DataBase Connection Pool), 처럼 공통된 객체를 여러개 생성해서 사용해야하는 상황에서 많이 사용. <br>

#### 단점
- 테스트하기가 힘들다
- private 생성자 때문에 상속이 어렵다 <br>
자바에서 상속하여 자식 클래스의 생성자가 호출 될 때 가장 먼저 부모 클래스의 기본 생성자가 호출됩니다. <br>
이는 super();로 호출하거나 생략이 가능합니다. 그런데 싱글톤을 위해서 생성자 접근범위를 private으로한다면 부모 생성자가 존재하지않아 상속이 불가능합니다. <br>

```
TMI로 JPA에서 지연로딩을 사용할때 프록시 객체를 사용합니다. 
그런데 프록시 객체를 생성하려면 프록시 객체가 원본 객체를 상속을 해야합니다. 
그래서 항상 엔티티에서 기본생성자를 protected나 public으로 만들어주는 패턴을 사용합니다. 
```

### 싱글턴 생성 방법 2가지
2가지 모두 생성자는 private으로 감춰두고 유일한 인스턴스에 접근할 수 있는 수단으로 public static멤버를 하나 마련 해 둡니다. <br>
Static이 붙은 변수 및 클래스는 프로그램 실행 시 메모리에 자동으로 생성되기 때문에. 인스턴스를 생성하지 않아도 사용할 수 있다. <br>


### 첫번째
```java
public class Singleton  {
    public static final Singleton INSTANCE = new Singleton();
    private Singleton(){};   
}
```


###  두 번째 : 정적 펙터리 메서드를 통해 인스턴스를 제공
```java
public class Singleton  {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton(){};
   
    private static Singleton getInstance(){
        return INSTANCE;
    }
}
두번째 지연 생성인 경우
public class Singleton {
    private static Singleton INSTANCE = null;
    private Singleton(){};
    public static Singleton getInstance(){
        if(INSTANCE == null){
            synchronized (Singleton.class){
                if(INSTANCE == null)
                    INSTANCE = new Singleton();
            }
        }
        return INSTANCE;
    }
}
```


#### 두번째 방법의 장점
첫번째, api를 변경하지 않고 클래스를 비싱글톤으로 만들수 있다는 장점이 있다. 
``` java
public class Singleton  {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton(){};   
    private static Singleton getInstance(){
        return new Singleton(); // INSTANCE -> new SingleTon()
    }
}
```
두번째, 정적 펙터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점이다. <br>
제네릭 싱글톤 팩토리는 디자인 패턴 중 하나인 싱글톤(Singleton) 패턴을 제네릭(Generic)으로 구현한 디자인 패턴입니다. 

```java
public class GenericSingletonFactory<T> {
    private static Map<Class<?>, Object> instances = new HashMap<>();

    public T getInstance(Class<T> type) {
        if (!instances.containsKey(type)) {
            try {
                T instance = type.newInstance();
                instances.put(type, instance);
            } catch (InstantiationException | IllegalAccessException e) {
                e.printStackTrace();
            }
        }
        return type.cast(instances.get(type));
    }
}
```

세 번째 장점은 정적 펙터리의 메서드 참조를 공급자로 사용할 수 있다는 점이다. 
```java
import java.util.function.Supplier;

public class Singleton  {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton(){};  
    private static Singleton getInstance(){
        return INSTANCE;
    }
}

public class Main {
    public static void main(String[] args) {
        // 정적 팩토리 메서드 참조를 사용하여 Supplier 생성
        Supplier<MyClass> supplier = Singleton::getInstance;

        // Supplier를 사용하여 Singleton 인스턴스를 생성
        Singleton instance = supplier.get();
    }
}
```

>#### 이러한 장점들이 필요하지 않다면 public필드 방식이 좋다고 합니다.


### 직렬화
위에서 말했던 2가지 방식으로 싱글턴 클래스를 생성한다고 해도 문제점이 있습니다. <br>
싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다. <br>
추가적인 readResolve메서드를 제공해야 싱글턴이 보장이됩니다. 그렇지 않으면 직렬화된 인스턴스를 역직렬화할 때 마다 새로운 인스턴스가 만들어 지기 떄문입니다. <br>


#### 싱글턴을 만드는 세 번째 방법은 원소가 하나인 열거 타입을 선언하는 것이다
```java
public enum OrderStatus {
    INIT("주문생성"),
    CANCELED("주문취소"),
    PAYMENT_COMPLETED("결제완료"),
    PAYMENT_FAILED("결제실패"),
    RECEIVED("주문접수"),
    COMPLETED("처리완료");
}
```
어떻게 보면 결론이라고 생각했습니다. <br>
>### 대부분 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다. 
단 주의점은 Enum외의 클래스를 상속해야 한다면 사용할 수 없다. <br>
(열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다) <br>
아래 코드는 열거 타입이 다른 인터페이스를 구현하도록 만든 코드입니다 <br>

```java
// 인터페이스를 이용해 확장 가능 Enum Type을 흉내냈다.
public interface Operation {
    double apply(double x, double y);
}
public enum BasicOperation implements Operation {
    PLUS("+"){
        public double apply(double x, double y) {return x + y;}
    },
    MINUS("-"){
        public double apply(double x, double y) {return x - y;}
    },
    TIMES("*"){
        public double apply(double x, double y) {return x * y;}
    },
    DIVIDE("/"){
        public double apply(double x, double y) {return x / y;}
    };
    private final String symbol;
    BasicOperation(String symbol){
        this.symbol = symbol;
    }
    @Override
    public String toString() {
        return symbol;
    }
}
```

### 개인적 의견
개인적 의견 처음에 이야기했듯이 싱글턴은 하나의 인스턴스를 재사용하는 경우에 사용한다고 했다. <br>
Enum으로 상수를 선언해서 재사용하면은 싱글턴을 올바르게 사용하는 것 같다. <br>
