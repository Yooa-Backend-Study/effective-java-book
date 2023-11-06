# 아이템 34 | int 상수 대신 열거 타입을 사용하라 

## 정수 열거 패턴의 단점
```java 
public static final int T_SHIRT_SMALL = 0;
public static final int T_SHIRT_MEDIUM = 1;
public static final int T_SHIRT_LARGE = 2;

public static final int PANTS_SMALL = 0;
public static final int PANTS_MEDIUM = 1;
public static final int PANTS_LARGE = 2;
```
1. 타입 안정성을 보장할 수 없으며 표현력도 좋지 않다.
    ```
    T-SHIRT_SMALL == PANTS_SMALL //true  
   ```
   ```java 
   public static final int ELEMENT_MERCURY = 0;
   public static final int PLANET_MERCURY = 0;
   ```
   
2. 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.


3. 문자열로 출력하기 까다롭다.
   ```java 
   public static final int T_SHIRT_SMALL = 0;
   System.out.println(T_SHIRT_SMALL); //0
   ```

4. 상수가 몇 개?

   ...

---
## 열거 타입 (Enum Type)
### 1. 열거 타입의 특징
1. 완전한 형태의 `클래스`, 상수 하나당 자신의 인스턴스를 하나씩 만들어 ___public static final___ 필드로 공개
   
   > 힙(heap) 메모리에 저장되며, 각 상수들은 별개의 메모리 주소 값을 가짐
   > ```java
   > public enum Tshirt {
   >    SMALL,
   >    MEDIUM,
   >    LARGE
   > }
   > ```
   > 
   > ``` java
   > package item34;
   > public class Test {
   > 
   >    public static void main(String... args) {
   >       Tshirt size = Tshirt.SMALL;
   >       //주소값 비교
   >       System.out.println(size == Tshirt.SMALL); //true
   >       System.out.println(size == Tshirt.MEDIUM); //false
   >    }
   > } 
   > ```
   > <img width="880" alt="item34" src="https://github.com/Yooa-Backend-Study/effective-java-book/assets/78305392/db6c8fb8-84db-4eaf-8b9b-fbbca066b531">
   > 


2. 싱글톤 
3. 컴파일타임 타입 안정성을 제공
4. namespace 제공
   ```java
   public enum Tshirt {SMALL, MEDIUM, LARGE}
   public enum Pants {SMALL, MEDIUM, LARGE}
   ```

5. 임의의 메서드나 필드 추가 또는 인터페이스를 구현하게 할 수도 있음

+ 새로운 상수를 추가하거나 순서를 바꿔도 클라이언트가 다시 컴파일하지 않아도 됨
- 열거 타입의 상수가 제거되어도 제거된 상수를 참조하지 않는 클라이언트에는 아무 영향이 없음

## 2. 열거 타입 생성 시 주의 사항 
1. 필드는 private, package-private 으로 구현
2. 톱 레벨 클래스거나 해당 클래스의 멤버 클래스로 구현
   ```java 
   public class Test {
      private enum Tshirt {SMALL, MEDIUM, LARGE}
   ...
   }
   ```

## 3. 열거 타입이 제공하는 메서드
| method               | description                | type   |
|----------------------|----------------------------|--------|
| name()               | 열거 객체의 문자열을 리턴	            | String |
| ordinal()            | 열거 객체의 순번(0부터 시작)을 리턴	     | int    |
| compareTo()          | 열거 객체를 비교해서 순번 차이를 리턴	     | int    |
| valueOf(String name) | 문자열을 입력받아서 일치하는 열거 객체를 리턴	 | enum   |
| values()             | 모든 열거 객체들을 배열로 리턴	         | enum[] |

> `java.lang.Enum 클래스`
> 
> 오버라이딩 하지 못하는 메서드 4개
> 
> 1.clone()
> 
> 2.finalize()
> 
> 3.hashCode()
> 
> 4.equals()
>

**toString() 재정의 시 fromString도 고려!**

valueOf() 가 있음에도 불구하고 fromString을 정의할 때의 이점
```java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(
            toMap(Object::toString, e -> e));

public static Optional<Operation> fromString (String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol.toLowerCase(Locale.ROOT)));
}

---

System.out.println(Operation.fromString("Plus")); // `symbol` plus 정상 출력
```
1. 유연성

2. 에러 처리

## 4. 열거 타입 활용

### 데이터와 메서드를 갖는 열거 타입
```java 
enum Food {

    KOREAN("한식당", Arrays.asList("된장찌개", "삼겹살", "떡볶이")),
    CHINESE("중식당", Arrays.asList("짜장면", "짬뽕", "볶음밥")),
    JAPANESE("일식당", Arrays.asList("초밥", "돈까스", "우동"));

    private final String Store;
    private final List<String> menu;
    private final String pick;

    Food(String name, List<String> menus) {
        this.Store = name;
        this.menu = menus;
        this.pick = menu.get((int) (Math.random() * menu.size()));
    }

    public String getFood() throws Exception {
        return "[" + Store + "] " + pick;
    }
}

---

public class Test {
   public static void main(String... args)  {
       System.out.println(Food.CHINESE.getFood()); //[중식당] 볶음밥
   }
}

```
- 한 눈에 관계를 가시적으로 확인 가능
- 데이터 그룹화에 용이
- 단순화 가능

### 열거 타입 확장
상수마다 동작이 달라져야 하는 경우 
1. `switch`
   - 깨지기 쉬운 코드! 

2. `apply() 추상 메서드`
   : 상수별 메서드 구현
   ```java
   package item34;
   
   enum Operation {
       PLUS("+") {
           public double apply(double x, double y) {
               return x + y;
           }
       }, MINUS("-") {
           public double apply(double x, double y) {
               return x - y;
           }
       }, MULTI("*") {
           public double apply(double x, double y) {
               return x * y;
           }
       }, DIVIDE("/") {
           public double apply(double x, double y) {
               return x / y;
           }
       };
   
       // 클래스 생성자와 멤버
       private final String symbol;
   
       Operation(String symbol) {
           this.symbol = symbol;
       }
   
       // toString을 재정의하여 열거 객체의 매핑된 문자열을 반환하도록
       @Override
       public String toString() {
           return symbol;
       }
   
       // 열거 객체의 메소드에 사용될 추상 메소드 정의
       public abstract double apply(double x, double y);
   }
   
   ```
   
→ **람다식**
```java
package item34;

// 함수형 인터페이스 임포트
import java.util.function.DoubleBinaryOperator;

enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    MULTI("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final DoubleBinaryOperator op; // 람다식을 저장할 필드

    private final String symbol;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() { return symbol; }


    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}

---

public class Test {
   public static void main(String... args) throws Exception {
      Tshirt size = Tshirt.SMALL;

      double x = 10.0;
      double y = 5.0;

      for (Operation op : Operation.values()) {
         double result = op.apply(x, y);
         System.out.println(x + " " + op + " " + y + " = " + result);
      }
   }

}
```
### 전략 열거 타입 패턴
상수별 메서드 작성 시 일부 공통으로 쓰이는 코드를 공유하기 위해서는
1. 모든 상수 메서드에 중복 코드 작성
2. 상황별로 도우미 메서드를 나눠 각각을 도우미 메서드로 작성 후 상수마다 호출

   → 가독성 BAD
   → 오류 발생 가능성 UP
   → 유지보수 어려움 

**전략 열거 타입 패턴**
```java
enum PayrollDay {
   MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
   THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
   SATURDAY(WEEKEND), SUNDAY(WEEKEND),
   NEW_HOLIDAY(HOLIDAY);

   private final PayType payType;
   PayrollDay(PayType payType) { this.payType = payType; }

   double pay(int minutesWorked, int payRate) {
      return payType.pay(minutesWorked, payRate);
   }
   // 전략 열거 타입
   enum PayType {
      WEEKDAY {
         double overtimePay(int minsWorked, int payRate) {
            return minsWorked <= MINS_PER_SHIFT ? 0 :
                    (minsWorked - MINS_PER_SHIFT) * payRate / 2;
         }
      },
      WEEKEND {
         double overtimePay(int minsWorked, int payRate) {
            return minsWorked * payRate / 2;
         }
      },
      HOLIDAY {
         double overtimePay(int minsWorked, int payRate) {
            return (minsWorked * (payRate + (payRate / 2))) / 2;
         }
      };

      abstract double overtimePay(int mins, int payRate);
      private static final int MINS_PER_SHIFT = 8*60;

      double pay(int minsWorked, int payRate) {
         int basePay = minsWorked * payRate;
         return basePay + overtimePay(minsWorked, payRate);
      }
   }
}

---

public class Test {
   public static void main(String... args) throws Exception {

      // 월요일(평일)에 8시간 일한 경우
      double mondayPayment = PayrollDay.MONDAY.pay(8 * 60, 10000);
      System.out.println("Payment for Monday: " + mondayPayment); //Payment for Monday: 4800000.0

      // 토요일(주말)에 6시간 일한 경우
      double saturdayPayment = PayrollDay.SATURDAY.pay(6 * 60, 12000);
      System.out.println("Payment for Saturday: " + saturdayPayment); //Payment for Saturday: 6480000.0

      // 수요일(평일)에 11시간 일한 경우
      double wednesdayPayment = PayrollDay.WEDNESDAY.pay(11 * 60, 10000);
      System.out.println("Payment for Wednesday: " + wednesdayPayment); //Payment for Wednesday: 7500000.0

      // 휴가일 때 4시간 일한 경우
      double holidayPayment = PayrollDay.NEW_HOLIDAY.pay(6 * 60, 12000);
      System.out.println("Payment for Holiday: " + holidayPayment); //Payment for Holiday: 7560000.0


   }
}

```

1. 확장성

2. 중복 코드 제거

---

**하지만 switch가 더 유용할 경우?**
1. 다른 데이터 유형 또는 객체

2. 여러 조건에 따른 다양한 동작

3. 자바 17 switch 패턴 매칭


## 5. 결론 

열거타입은 객체이다 보니 메모리 할당 또는 가비지 컬렉션 측면에서의 오버헤드가 있을 수 있지만 !

1. 타입 안정성

2. 가독성

3. 확장성

4. 싱글톤

5. 내부 메서드와 필드를 가질 수 있음

위와 같은 장점을 가지므로 되도록 열거 타입을 사용하자 