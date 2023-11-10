## 📝 [아이템 36] 비트 필드 대신 EnumSet을 사용하라

### 비트 필드
```java
public class Text {
    public static final int BOLD = 1 << 0; // 첫 번째 비트 // 1 // 이진수 : 0001
    public static final int ITALIC = 1 << 1; // 두 번째 비트 // 2 // 이진수 : 0010
    public static final int UNDERLINE = 1 << 2; // 세 번째 비트 // 4 // 이진수 : 0100
    public static final int STRIKETHROUGH = 1 << 3; // 네 번째 비트 // 8 이진수 : 1000

    public void applyStyles(int styles) {
        // ...
    }
}
```

> 클래스 안에 상수로 표현할 때 쉬프트 연산을 사용해 비트로 표현하는 것. 이렇게 사용하면 상수 값들을 비트로 표현할 수 있음
>

**즉, 비트값으로 표현된 Enum들에게 OR 비트 연산자(|)를 넣어서 Enum 값들을 합쳐서 표현할 수 있게 됨.**
```java
text.applyStyles(BOLD | ITALIC); // 첫 번째와 두 번째 비트 OR 연산으로 3 , 01 | 10 -> 11
```  

### 비트 필드의 단점

```java
text.applyStyles(BOLD | ITALIC); // 첫 번째와 두 번째 비트 OR 연산으로 3 , 01 | 10 -> 11
```  

당연하게도 text.applyStyles(3) 메서드에 인자로 들어가는 값은 OR 비트 연산으로 3이 들어가게 됨.  

- 즉, applyStyles() 안에서 3이란 값으로 로직을 실행해야 하는데, 3이라는 숫자는 BOLD, ITALIC 이라는 정보를 가지고 있지 않다.     
  **해석하기가 어려워짐.**


게다가 Enum 개수가 늘어날 때마다, 그만큼 쉬프트 연산하는 비트값도 커지므로 자료형으로 int나 long을 선택할 지 정해야 함.

또한, 열거값들을 순회하고 싶다면? 다른 로직을 만들어야 합니다.

### 비트필드의 대안, EnumSet

> ✔️ **EnumSet 클래스** 는, 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.
>
> 또한, 1) Set 인터페이스를 완벽히 구현하며 2) 타입 안전하고 3) 다른 어떤 Set 구현체와도 함께 사용할 수 있다.
> 

- 하지만, EnumSet의 **내부는 비트 벡터로 구현되어 있으며** 원소의 개수가 64개 이하라면, long 변수 하나로 표현할 수 있어서 비트필드에 비견되는 성능을 보여준다.
- removeAll과 retailAll과 같은 대량 작업은 비트를 효율적으로 처리 할 수 있는 산술 연산을 사용하였다.


```java
public class Text {

    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public static void applyStyles(Set<Style> styles) {
        //...
    }

    public static void main(String[] args) {
        // ...
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC, Style.UNDERLINE));
    }
}
```
&nbsp;

### 결론
- 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.
- EnumSet 클래스가 비트 필드 수준의 명료함과 성능, 그리고 열거 타입의 장점까지 제공해준다.
- EnumSet의 유일한 단점은 자바 9까지는 아직 불변 EnumSet을 만들 수 없다는 것이다(자바 11까지도 수정되지 않았다).
- 명확성과 성능이 조금 희생되긴 하지만, EnumSet 대신 Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다.

