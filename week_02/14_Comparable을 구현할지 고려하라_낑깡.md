# 아이템13 | Comparable을 구현할지 고려하라


### 결론 
자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 Comparable을 구현한 덕분에 손쉽게 정렬 가능 

-> 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자!

---
### Comparable의 compareTo와 equals의 차이점
1. 순서 비교
   -  -1 (객체 < 주어진 객체), 0 (객체 == 주어진 객체), 1 (객체 > 주어진 객체)
2. 제네릭 (다양한 타입의 객체를 비교할 수 있으며, 타입 안전성을 보장할 수 있음)
3. 타입이 다른 객체를 신경 쓸 필요 없이 간단히 ClassCastException을 던진다.
---
### compareTo의 일반 규약과 equals의 규약
> compareTo의 일반규약
>1. 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과를 도출해야 한다. [반사성]
>2. 첫 번째가 두 번째보다 크고, 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다. [추이성]
>3. 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다. [대칭성]
>4. compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.


-> compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 세 성격을 충족해야 함을 뜻한다.

-> equals와 주의사항도 똑같다.

: 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지키지 못할 수 있다.
```java
public class Point implements Comparable<Point> {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public int compareTo(Point other) {
        // 거리를 기준으로 비교
        double thisDistance = Math.sqrt(x * x + y * y);
        double otherDistance = Math.sqrt(other.x * other.x + other.y * other.y);

        return Double.compare(thisDistance, otherDistance);
    }
}

```

```java
public class ColoredPoint extends Point implements Comparable<ColoredPoint> {
    private String color;

    public ColoredPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public int compareTo(ColoredPoint other) {
        // 어떤 값을 기준으로 비교해야 하는가?
        // x, y, color 중 어떤 것을 선택할지 애매함
        // compareTo 규약을 위반할 가능성이 높음
    }
}

```
-> 확장하는 대신 `컴포지션`을 사용하자 !

(💡다음과 같은 방식으로 Comparable을 구현한 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가해봄 )
```java
package item14;

public class Point implements Comparable<Point> {
    int x;
    int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public int compareTo(Point other) {
        // 거리를 기준으로 비교
        double thisDistance = Math.sqrt(x * x + y * y);
        double otherDistance = Math.sqrt(other.x * other.x + other.y * other.y);

        return Double.compare(thisDistance, otherDistance);
    }
}

```
```java
package item14;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ColoredPoint extends Point {
    private String color;

    public ColoredPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public int compareTo(Point other) {
        // 먼저 상위 클래스(Point)의 compareTo를 호출하여 기존 비교를 수행
        int superComparison = super.compareTo(other);

        // 새로운 값 컴포넌트(color)를 고려하여 추가 비교
        if (superComparison != 0) {
            return superComparison; // 상위 비교 결과가 0이 아니면 반환
        }

        // color 기준으로 추가 비교
        return String.CASE_INSENSITIVE_ORDER.compare(this.color, ((ColoredPoint) other).color);
    }

    @Override
    public String toString() {
        return "ColoredPoint{" +
                "color='" + color + '\'' +
                ", x=" + x +
                ", y=" + y +
                '}';
    }

    public static void main(String[] args) {
        ColoredPoint a = new ColoredPoint(1, 2, "a");
        ColoredPoint b = new ColoredPoint(2, 1, "b");

        List<ColoredPoint> list = new ArrayList<>();
        list.add(a);
        list.add(b);

        System.out.println("list1 : " + a.compareTo(b));

        Collections.sort(list);
        System.out.println(list);

        List<ColoredPoint> list2 = new ArrayList<>();
        list2.add(b);
        list2.add(a);

        System.out.println("list2 : " + b.compareTo(a));
        Collections.sort(list2);
        System.out.println(list2);

    }
}
```

> list1 : -1
> 
>[ColoredPoint{color='a', x=1, y=2}, ColoredPoint{color='b', x=2, y=1}]
> 
>list2 : 1
> 
>[ColoredPoint{color='a', x=1, y=2}, ColoredPoint{color='b', x=2, y=1}]
> 

equals 규약에서는 위반되는 예제이지만, compareTo 규약에는 위반되지 않는게 아닌가? ...

### compareTo 메서드 작성 요령
1. **입력 인수의 타입을 확인하거나 형변환할 필요가 없다.** 
   - 타입을 인수로 받는 제네릭 인터페이스이므로 인수 타입은 컴파일 타임에 정해진다.
   - 인수의 타입이 잘못됐다면 컴파일 자체가 되지 않으며 null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.
   
**2. 객체 참조 필드를 비교할 시 compareTo 메서드를 재귀적으로 호출한다.**

**3. compareTo에서 관계 연산자 (>, <) 사용하지 않기**
   - 자바 7부터 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드 compare 이용하기
   ```java 
        Integer.compare(a, b) 
   ```

### Comparator + 비교자 생성 메서드 => 메서드 연쇄 방식
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 할 때 사용
- 자바 8에서는 Comparator 인터페이스가 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
   - 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는 데 활용할 수 있다.
- 약간의 성능 저하의 이유는 ?


```java
package item14;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class Person implements Comparable<Person> {
    private static final Comparator<Person> COMPARATOR =
            Comparator.comparing((Person person) -> person.name)
                    .thenComparingInt(person -> person.age)
                    .thenComparing(person -> person.gender);

    private String name;
    private int age;
    private String gender;

    public Person(String name, int age, String gender) {
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    @Override
    public int compareTo(Person o) {
        return COMPARATOR.compare(this, o);
    }

    @Override
    public String toString() {
        return "{" +
                "name =" + name +
                ", age =" + age +
                ", gender =" + gender +
                '}';
    }

    public static void main(String[] args) {
        Person person1 = new Person("eunseo", 16, "m");
        Person person2 = new Person("eunseo", 8, "m");

        List<Person> list = new ArrayList<>();
        list.add(person1);
        list.add(person2);

        Collections.sort(list);

        System.out.println(list);

    }
}

```

>[{name =eunseo, age =8, gender =m}, {name =eunseo, age =16, gender =m}]

### 객체 참조용 비교자 생성 메서드 
Comparator 인터페이스를 사용하여 객체를 비교할 때 도움을 주는 메서드

이러한 메서드는 Java 8부터 추가되었으며, 객체를 비교하는 데 사용되며 람다 표현식 또는 메서드 참조를 생성하여 다양한 비교 작업을 수행할 수 있다.

```java 
Comparator<Item> comp = new Comparator<Item>() {
            public int compare(Item o1, Item o2) {
                return o1.c - o2.c;
            }
        };
```

-> 정수 오버플로우를 일으킬 수 있음 !!!

**정적 compare 메서드를 활용한 비교자**
```java 
    Comparator<Item> comp = new Comparator<Item>() {
        public int compare(Item o1, Item o2) {
                return Integer.compare(o1.a, o1.b);
                }
        };
```
**비교자 생성 메서드를 활용한 비교자**
```java 
    Comparator<Item> comp = Comparator.comparingInt(item -> item.b);
```

**getter + 람다식 사용**
```java 
    Comparator<Item> comp = Comparator.comparingInt(Item::getB);
    
    Comparator.comparingInt(Item::getB).reversed().thenComparingInt(Item::getC); 
```

---

Comparable을 구현하고, 컬렉션과 함께 사용하여 손쉽게 값을 비교하고 정렬하자

compareTo 메소드를 사용할 때는 박싱된 기본 타입 클래스가 제공하는 정적 compare 메소드를 이용하고 상황에 맞는 비교자를 사용하자

---

