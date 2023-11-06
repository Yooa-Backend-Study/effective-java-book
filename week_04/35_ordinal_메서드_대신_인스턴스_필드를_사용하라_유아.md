# [아이템 35] ordinal 메서드 대신 인스턴스 필드를 사용하라

## `ordinal` 메서드

열거 타입 상수는 하나의 정숫값에 대응된다. 그 해당 상수가 열거 타입에서 몇 번째 위치인지를 반환하는 `ordinal` 메서드 제공(0부터 시작)

```java
public enum Item35 {
    JENNIE, AKMU, ORANGE_CARAMEL, AESPA;

    public int numberOfMember() { return ordinal() + 1; }
}
```

유지보수가 어려운 코드.

- 상수 선언 순서를 바꾸면 오동작
- 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다. (4인 그룹 블랙 핑크를 추가할 수 없다.)
- 값을 중간에 비워둘 수 없음
  - 갑자기 세븐틴을 추가하면 13을 반환해야 하는데, 이를 위해 5~12까지의 더미 상수를 함께 추가해야 하는 상황
  - 코드가 깔끔하지 못하고 실용성이 떨어짐

열거 타입 상수에 견결된 값은 ordinal 메서드가 아닌 필드에 저장하자.

```java
public enum Item35 {
    JENNIE(1), AKMU(2), ORANGE_CARAMEL(3), AESPA(4), SEVENTEEN(13);

    private final int numberOfMembers;
    Item35(int size) { this.numberOfMembers = size; }
    public int numberOfMember() { return numberOfMembers; }
}
```

> Enum API 문서를 보면..
>
> ```java
>
> /**
>  * Returns the ordinal of this enumeration constant (its position
>  * in its enum declaration, where the initial constant is assigned
>  * an ordinal of zero).
>  *
>  * Most programmers will have no use for this method.  It is
>  * designed for use by sophisticated enum-based data structures, such
>  * as {@linkjava.util.EnumSet} and {@linkjava.util.EnumMap}.
>  *
>  *@returnthe ordinal of this enumeration constant
>  */
> public final int ordinal() {
>     return ordinal;
> }
> ```
>
> Most programmers will have no use for this field. It is designed for use by sophisticated enum-based data structures, such as
> {@linkjava.util.EnumSet} and {@linkjava.util.EnumMap}.
>
> **대부분의 프로그래머는 이 메서드를 사용하지 않는다. EnumSet이나 EnumMap에서 사용하기 위해 만들어진 메서드이다.**

공식 문서에서 특별한 용도를 제외하고 사용하지 말라니 쓰지 말아야겠다.

### EnumSet

```java
EnumSet.of(TestStatus.WAIT, TestStatus.COMPLETED);// 원하는 Enum들을 넣어서 생성
EnumSet.noneOf(TestStatus.class);// 빈 공간으로 생성
EnumSet.allOf(TestStatus.class);// 해당 Enum 에 모든 것을 생성
```

- EnumSet 내부 표현은 비트 벡터로 표현된다. (상수 개수가 64개 이하라면 long변수 하나로 표현한다.)
- 여기서 EnumSet 은 Enum 상수가 선언된 순서, 즉 ordinal() 메서드의 반환된 순서로 순회한다.

## JPA Enum 필드

JPA는 Enum 필드도 테이블 컬럼과 매핑해준다. 그때 사용하는 어노테이션이 `@Enumerated`

여기서 EnumType을 설정해줘야 하는데, `ORDINAL`과 `STRING`이 있다.

- ORDINAL은 ordinal() 메서드
- STRING은 name() 메서드

상수 선언 순서를 바꾸면 오동작하게 될 것이고, 원래 있던 데이터들이 모두 꼬이게 되는 치명적인 문제가 발생할 수 있다. 그러기 때문에 JPA에서 @Enumerated 의 ORDINAL도 유의해서 사용해야 한다.
