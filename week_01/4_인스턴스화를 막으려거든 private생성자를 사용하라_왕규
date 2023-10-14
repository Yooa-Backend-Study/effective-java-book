>### 인스턴화를 막으려 거든 private 생성자를 사용하라
java.lang.Math와 같이 메서드들을 모아 놓은 클래스를 만들어야 하는 상황이 올 수 있습니다. 예를들어 다음과 같이 반복적으로 사용되는 계산식이 있습니다
```java
public final class BigDecimalCalculator {
    private BigDecimalCalculator(){};
    >
    public String add(BigDecimal n1, BigDecimal n2){
        return n1.add(n2).multiply(n2).toPlainString();
    }
}
```

그래서 이러한 클래스들은 인스턴스로 만들어서 사용할려고 설계한게 아닙니다.
그러나 기본 생성자를 명시하지 않는다면은 컴파일러가 항상 기본생성자를 public으로 만들어 줍니다. 그래서 의도치 않게 인스턴스화할 수 있게 됩니다.

추상클래스는 인스턴스를 생성할 수 없기 때문에 추상클래스를 사용하려고 하지만 추상클래스도 인스턴스화를 막을 수 없습니다. 하위클래스를 만들어 상속한다면 자동으로 인스턴스화가 됩니다.

다행히 인스턴스화를 막는 방법은 간단하빈다.
명시적으로 private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있습니다.
Lombok을 사용하면 다음과 같이 편리하게 됩니다.
```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class ErrorMessages
```
명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, 이를 private으로 선언했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀버립니다.






```java
public enum ErrorMessages {
    EXISTS_EMAIL("주문생성"),
}
``

Enum으로는 다음과같이 class에서 EXISTS_EMAIL 필드를 표현이 가능합니다.
Enum으로는 주로 상태를 표현에 정형화 되어 있습니다. 
그래서 Enum으로 메소드를 만들 수 있지만, 클래스로 만들어서 사용하기도 합니다.java.lang.Math와 메서드 들을 

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class ErrorMessages {
    // 회원가입
    public static final String EXISTS_EMAIL = "이미 존재하는 이메일입니다";
}
```
