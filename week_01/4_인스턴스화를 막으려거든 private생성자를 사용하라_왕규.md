>### 인스턴화를 막으려 거든 private 생성자를 사용하라
java.lang.Math와 같이 메서드들을 모아 놓은 클래스를 만들어야 하는 상황이 올 수 있습니다. 예를 들어 다음과 같이 반복적으로 사용되는 계산식이 있습니다
```java
public final class BigDecimalCalculator {
    private BigDecimalCalculator(){};
    >
    public String add(BigDecimal n1, BigDecimal n2){
        return n1.add(n2).multiply(n2).toPlainString();
    }
}
```

그래서 이러한 클래스들은 인스턴스로 만들어서 사용할려고 설계한 게 아닙니다.
그러나 기본 생성자를 명시하지 않는다면은 컴파일러가 항상 기본 생성자를 public으로 만들어 줍니다. 그래서 의도치 않게 인스턴스화할 수 있게 됩니다.

추상 클래스는 인스턴스를 생성할 수 없기 때문에 추상 클래스를 사용하려고 하지만 추상 클래스도 인스턴스화를 막을 수 없습니다. <br>
하위 클래스를 만들어 상속한다면 자동으로 인스턴스화가 됩니다. <br>
<br>
다행히 인스턴스화를 막는 방법은 간단합니다.<br>
명시적으로 private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있습니다. <br>
Lombok을 사용하면 다음과 같이 편리하게 됩니다.
```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class ErrorMessages
```
명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, 이를 private으로 선언했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀버립니다. <br>




### 개인적 의견
공통 사용되는 메서드는 final 클래스를 사용해서 상속을 못 하게 만들고 private 생성자로 개발합니다. <br>
Enum도 메서드를 만들 수 있지만 주로 상수 표현에 자주 사용되기 때문에 상수 표현에는 Enum, 공통 메서드 표현에는 class를 이용합니다.

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class ErrorMessages {

    public static void hello(){
        System.out.println("hello");
    }

    // x 아래와 같은 코드는 Enum으로 빼자
    public static final String EXISTS_EMAIL = "이미 존재하는 이메일입니다";
}

```java
public enum ErrorMessages {
    EXISTS_EMAIL("이미 존재하는 이메일입니다"),
}
``
