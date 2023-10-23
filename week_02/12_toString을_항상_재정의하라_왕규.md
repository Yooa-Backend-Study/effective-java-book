이번 주제는 항상 toString을 사용하려면 재정의하라는 문장을 많이 본적이 있을 것입니다.


### Object 클래스의 toString 메소드의 구현
기본적으로 클래스는 가장 상위인object를 상속하게 됩니다. 그래서 앞에 보았던 equals나 hashCode과 같이 toString도 재정의를 염두에 두고 설계된 것이라 재 정의 시 지켜야 하는 규칙이 있습니다.
먼저 기본적인 Object의 toString입니다.
```java
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```


### toString은 왜 재정의 해야할가??
대표적 2가지 이유 
1. 디버깅 하기 편리하다 :  디버깅 시 객체의 내부 상태를 쉽게 확인할 수 있습니다, 로그에 기록할 때도 편리하다
toString정의하기 전 <br>
![](https://velog.velcdn.com/images/cwangg897/post/7a7c418d-c88a-4331-a2f3-d8de30d936fe/image.png)
정의하고 난 후 <br>
![](https://velog.velcdn.com/images/cwangg897/post/9c22d478-7b49-4f12-b715-575087586b27/image.png)
2. print나 log로 출력했을때 보기 편합니다


#### 그렇다면 toString은 어떻게 재정의해야 할까?
```
toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋습니다. 
전화번호면 Jenny=010-1234-1234와 같은 예시로 표현하는 게 좋다고 합니다.
만약 표현해야 할 정보가 많다면 요약해서 정보를 표현하는 게 좋습니다.
```

#### toString구현한다면 포맷 문서화할지 선택해야 한다
포맷 문서화란?
gpt - Java에서 "toString 포맷을 문서화한다"는 것은 toString 메서드의 반환 값의 형식 및 구조를 명시적으로 문서화하고 설명하는 것을 의미합니다. 
이것은 주로 API 문서 또는 주석을 통해 수행됩니다. <br>
 
- 장점 : 포맷을 문서화 한다면 표준적이고 사람이 읽을 수 있게 됩니다. 협업한다면 개발자들이 동일한 포맷을 따르게 되어 일관성이 생길것이고 결과를 이해하기 편해질것입니다.
- 단점 : 포맷을 한번 명시하고 포맷을 명시한 클래스가 많이 사용된다면 평생 한번설정한 포맷에 얽메이게 됩니다. 포맷을 바꾼다면 모두 엉망이 될것입니다. 



명시하는 경우
```java
/**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
     *
     * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
     * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
     * 전화번호의 마지막 네 문자는 "0123"이 된다.
     */
    @Override public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }
```

>#### 포맷을 명시한다면 정적 펙터리나 생성자를 함께 제공하면 더 좋다
```java
public class MyDate {
    private int year;
    private int month;
    private int day;

    public MyDate(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }
    public static MyDate fromString(String dateStr) {
        // 문자열을 파싱하여 MyDate 객체로 변환
        String[] parts = dateStr.split("-");
        int year = Integer.parseInt(parts[0]);
        int month = Integer.parseInt(parts[1]);
        int day = Integer.parseInt(parts[2);
        return new MyDate(year, month, day);
    }

    @Override
    public String toString() {
        // 명시한 포맷으로 MyDate를 문자열로 변환
        return String.format("%04d-%02d-%02d", year, month, day);
    }

    public static void main(String[] args) {
        MyDate date = new MyDate(2023, 10, 21); // (생성자)

        // 객체를 문자열로 변환
        String dateStr = date.toString();
        System.out.println(dateStr); // 출력: "2023-10-21"

        // 문자열을 객체로 변환 (펙터리 메서드)
        MyDate parsedDate = MyDate.fromString("2023-10-21");
        System.out.println(parsedDate); // 출력: "2023-10-21"
    }
}
```


안하는 경우
```java
/**
         * 이 약물에 관한 대략적인 설명을 반환한다.
         * 다음은 이 설명의 일반적인 형태이나,
         * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
         * 
         * "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
         */
	@Override public String toString() {
```
이렇게 한다면 포맷에 맞춰 코딩하는 경우를 피할 수도 있어 포맷이 바뀌어 피해를 볼 경우가 적어집니다




#### 정적 유틸리티는 toString제공할 이유가 없다
정적 유틸리티 클래스(static utility class)는 주로 인스턴스 메서드를 갖지 않고, 정적 메서드로만 구성된 클래스입니다. <br>
```
이유들
1. 정적 유틸리티 클래스는 객체를 생성하지 않으며 인스턴스 변수를 갖지 않습니다. 
toString 메서드는 주로 객체의 내부 상태를 문자열로 표현하는 데 사용되며, 
객체의 상태를 나타내는 데 관련이 없는 정적 클래스에는 toString을 사용할 필요가 없습니다.

2. 정적 클래스의 목적은 메서드 호출을 통해 기능을 제공하는 데 중점을 둡니다.
객체의 상태를 관리하거나 문자열 표현을 다루는 것이 주요 목적이 아니라면 toString을 구현할 필요가 없습니다.

```

#### enum도 왜 toString제공 안하는 이유
이처럼 enum은 이미 toString 메서드를 가지고 있으며, 기본적으로 name() 메서드도 제공합니다. 
따라서 enum을 문자열로 표현하는 데 문제가 없으며, 추가적인 toString 메서드를 정의할 필요가 없습니다.
```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

public class EnumExample {
    public static void main(String[] args) {
        Day today = Day.WEDNESDAY;

        // toString 메서드를 사용하여 문자열로 변환
        String todayString = today.toString();
        System.out.println(todayString); // 출력: "WEDNESDAY"

        // name 메서드를 사용하여 enum 상수의 이름을 얻음
        String todayName = today.name();
        System.out.println(todayName); // 출력: "WEDNESDAY"
    }
}
```

### JPA에서의 순환참조 경우

#### 1. 양방향 연관관계에서 그대로 출력하는 경우
```java
System.out.println(phone.getLMember()); 코드 실행
```
toString 메서드를 오버라이드하지 않고 롬복 라이브러리로 생성하면 순환 참조로 이어질 수 있습니다.
JPA에서는 양방향으로 연결된 Entity끼리 조회하는 경우 서로의 정보를 순환하면서 조회하다가 stackoverflow가 발생하게 된다.
자동 생성되는 toString은 재정의해야 합니다.


#### 2. Entity를 그대로 반환(json으로 return)하는 경우
toString뿐만아니라 json생성 라이브러리에서도 이런 문제가 발생합니다
Spring Boot 는 Controller에 @ResponseBody 선언 시 Object 를 JSON 형태로 직렬화하기 위해 HttpMessageConverters 에서 jackson Library 활용하기 때문에 발생하게 됩니다
```
Author 엔티티를 JSON 형태로 직렬화하는 과정에서, Author 엔티티가 참조하고 있는 Book 엔티티를 조회하게 된다.
Book 엔티티를 조회하는 과정에서, Book 엔티티가 참조하고 있는 Author 엔티티를 조회한다.
다시 Author 엔티티를 조회하는 과정에서, Author 엔티티가 참조하고 있는 Book 엔티티를 조회... 다시 Book 엔티티를 조회..
이렇게 위 과정이 끊임없이 반복되며 두 Entity는 계속 서로를 참조하다 결국 StackOverflowError 를 발생시키게 된다.
```


#### 해결방법
1. Lombok 에 @Data 안에는 @ToString이 있으므로 get,set을 사용할거라면@Getter, @Setter 을 사용하자
2. Object 의 toString을 재정의후 참조하는 객체를 지운다
3. DTO를사용하자


#### 개인적 의견
개발하다가 print로 직접 확인하거나 log로 확인하거나 디버깅으로 확인할 때 편할려고 사용하라는 용도로 만들어진 거 같습니다.
toString도 하나의 문서로 활용될 수 있다고 느껴졌습니다.
