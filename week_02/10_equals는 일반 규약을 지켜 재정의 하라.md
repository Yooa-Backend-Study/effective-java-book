## 아이템 10. equals는 일반 규약을 지켜 재정의 하라

> equals를 다 구현했다면 세 가지만 자문해보자.
>
> **대칭적인가? 추이성이 있는가? 일관적인가?**
>
> equals 메서드를 재정의하지 않고 그냥 두면, 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.   


#### 1. equals를 재정의 하면 안 되는 경우
- **각 인스턴스가 본질적으로 고유할 때**   
   값 클래스(Integer나 String처럼 값을 표현하는 클래스)가 아닌 동작하는 개체를 표현하는 클래스   ex) Thread  
- **인스턴스의 '논리적 통치성'을 검사할 일이 없을 때**  
   ex) java.util.regax.Pattern은 equals를 재정의해 두 Pattern의 정규표현식을 비교

- **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 때**  
   ex) Set은 AbstractSet이 구현한 equals를 상속, List는 AbstractList, Map은 AbstractMap

- **클래스가 private이나 package-private이고 equals를 호출할 일이 없을 때**  
   아래와 같이 구현해 equals가 실수로라도 호출되는 걸 막을 수 있다.
``` java
@Override public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```
##
#### 2. equals를 재정의 해야 하는 경우

객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아닌 **'논리적 동치성'을 확인**해야 하는데,   
**상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않았을 때 (주로 값 클래스)**  

ex) 두 값 객체를 ```equals```로 비교하는 경우, 객체가 같은지가 아니라 값이 같은지를 알고싶을 것이다. 

```equals```가 논리적 동치성을 확인하도록 재정의하면, 값 비교는 물론 ```Map```의 키와 ```Set```의 원소로 사용 가능.   

**but, 값 클래스여도, 같은 인스턴스가 둘 이상 만들어지지 않는 인스턴스 통제 클래스라면 재정의하지 않아도 됨.**

##
#### 3. equals 메서드 재정의 일반 규약: 동치관계
**동치 클래스(equivalent class): 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산**    

→ equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 교환이 가능해야 한다.

✔️ **반사성(reflexivity)** 
: ```null```이 아닌 모든 참조 값 x에 대해, ```x.equals(x)```는 **true**다. 

✔️ **대칭성(symmetry)**  
: ```null```이 아닌 모든 참조 값 x, y에 대해, ```x.equals(y)```가 **true**면 ```y.equals(x)```도 **true**다.  

✔️**추이성(transitivity)**
: ```null```이 아닌 모든 참조 값 x, y, z에 대해, ```x.equals(y)```가 **true**이고, ```y.equals(z)```도 **true**면 ```x.equals(z)```도 **true**다.  

✔️ **일관성(consistency)**  
: ```null```이 아닌 모든 참조 값 x, y에 대해, ```x.equals(y)```를 **반복해서 호출**하면 항상 true이거나 false다.  

☑️ **null-아님** 

: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다. 

✔️ **반사성(reflexivity)**  

객체가 자기 자신과 같아야 한다.
```java
public class ProgrammingLanguage{
  private String name;

  public Fruit(String name){
    this.name = name;
  }

  public static void main(){
    Set<ProgrammingLanguage> set = new HashSet<>();
    ProgrammingLanguage language = new ProgrammingLanguage("java");
    set.add(language);
    System.out.println(set.contains(language)); // false일 경우, 반사성을 만족하지 못하는 경우이다.
  }
}
```
✔️ **대칭성(symmetry)**  

두 객체는 서로에 대한 동치 여부에 **똑같이 답해야** 한다.   

잘못된 코드 - 대칭성 위배!

```java
// 대칭성을 위반한 클래스
public final class CaseInsensitiveString{
  private final String s;

  public CaseInsensitiveString(String s){
    this.s = Obejcts.requireNonNull(s);
  }

  // 대칭성 위배!
  @Override public boolean equals(Object o){
    if(o instanceof CaseInsensitiveString)
      return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    if(o instanceof String) // 한방향으로만 작동한다.
      return s.equalsIgnoreCase((String) o);
    return false;
  }
}
```  

문제는 ```CaseInsensitiveString```의 ```equals```는 ```String```을 알고 있지만,   

```String```의 ```equals```는 ```CaseInsensitiveString```의 존재를 모른다는 데 있다.    

대칭성을 명백히 위반한다.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s); // true
s.equals(cis); // false
```


**해결 - CaseInsensitiveString끼리만 비교하도록 한다.**

```java
//대칭성을 만족하게 수정
@Override public boolean equals(Object o){
  return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s); 
}
```
✔️ **추이성(transitivity)**  

**첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같아면, 첫 번째 객체와 세 번째 객체도 같아야 한다.**  

상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하며 equals를 재정의할 때 자주 발생하는 문제다.

```java
public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override public boolean equals(Object o) {
		if(!o instanceof Point)
			return false;
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}
public class ColorPoint extends Point {
	private final Color color;
	
	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}

	...
}
```
**1. 잘못된 코드 - 대칭성 위배!**

```java
    @Override public boolean equals(Object o) {
    	if(!o instanceof ColorPoint)
    		return false;
    	return super.equals(o) && ((ColorPoint) o).color == color;
    }
    public static void main(){
      Point p = new Point(1,2);
      ColorPoint cp = new ColorPoint(1,2, Color.RED);
      p.equals(cp);    // true
      cp.equals(p);    // false
    }
```

ColorPoint의 equals는 입력 매개변수의 클래스 종류가 다르다며 매번 false만 반환할 것이다.

**2. 잘못된 코드 - 추이성 위배!**

```java
    @Override public boolean equals(Obejct o){
      if(!(o instanceof Point))
        return false;
      if(!(o instanceof ColorPoint))
        return o.equals(this);
      return super.equals(o) && ((ColorPoint) o).color == color;
    }
    public static void main(){
      ColorPoint p1 = new ColorPoint(1,2, Color.RED);
      Point p2 = new Point(1,2);
      ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
      p1.equals(p2);    // true 
      p2.equals(p3);    // true 
      p1.equals(p3);    // false
    }
```
```p1.equals(p2);```와 ```p2.equals(p3);```는 **true를 반환**하는데, ```p1.equals(p3);```는 **false를 반환**해 추이성에 위배된다.  이 방식은 **무한 재귀**에 빠질 위험도 있다.

  ``` java
    //SmellPoint.java의 equals
    @Override public boolean equals(Obejct o){
      if(!(o instanceof Point))
        return false;
      if(!(o instanceof SmellPoint))
        return o.equals(this);
      return super.equals(o) && ((SmellPoint) o).color == color;
    }
    public static void main(){
      ColorPoint p1 = new ColorPoint(1,2, Color.RED);
      SmellPoint p2 = new SmellPoint(1,2);
      p1.equals(p2);
      // 1. ColorPoint의 equals: 2번째 if문 때문에 SmellPoint의 equals로 비교
      // 2. SmellPoint의 equals: 2번째 if문 때문에 ColorPoint의 equals로 비교
      // 3. 1~2 무한 재귀로 인한 StackOverflow Error
    }
```    
구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.  

**3. 잘못된 코드 - 리스코프 치환 원칙 위배!** 

그렇다고 ```instanceof``` 검사 대신 ```getClass``` 검사를 하라는 것은 아니다.

```java
    @Override public boolean equals(Object o){
      if(o == null || o.getClass() != getClass())
        return false;
      Point p = (Point) o;
      return p.x == x && p.y == y;
    }
```
위의 코드는 **같은 구현 클래스의 객체와 비교할 때**만 ```true```를 반환한다.  

> 리스코프 치환 원칙: 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.  

= Point의 하위 클래스는 여전히 Point이므로 어디서든 Point로써 활용될 수 있어야 한다.

## 

#### 해결 1 - 상속 대신 컴포지션을 사용하라 (```equals``` 규약을 지키면서 값 추가하기)

``` java
public class ColorPoint{
  private final Point point;
  private final Color color;

	public ColorPoint(int x, int y, Color color) {
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color);
	}

	/* 이 ColorPoint의 Point 뷰를 반환한다. */
  public Point asPoint(){ // view 메서드 패턴
    return point;
  }

  @Override public boolean equals(Object o){
    if(!(o instanceof ColorPoint)){
      return false;
    }
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```

> 컴포지션: 기존 클래스가 새로운 클래스의 구성 요소로 쓰인다.
>
> **기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.**
>
> 컴포지션을 통해 새 클래스의 인스턴스 메서드들은 **기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환한다.**   


```Point```를 상속하는 대신 ```Point```를 ```ColorPoint```의 *private* 필드로 두고, ```ColorPoint```와 같은 위치의 일반 ```Point```를 반환하는 뷰(view 메서드)를 *public*으로 추가하는 식이다.

- ```ColorPoint``` vs. ```ColorPoint```: ```ColorPoint```의 ```equals```를 이용하여 color값까지 모두 비교  

- ```ColorPoint``` vs. ```Point```: ```ColorPoint```의 ```asPoint```를 이용하여 ```Point```로 바꿔, ```Point```의 ```equals```를 이용해 x, y비교  

- ```Point``` vs. ```Point```: Point의 equals를 이용해 x, y값 모두 비교  

ex) java.sql.Timestamp: java.util.Date 확장 후 nanoseconds 필드 추가.  

→ Timestamp의 equals는 대칭성을 위배하며, Date와 섞어 쓸 때 엉뚱하게 동작할 수 있다.

##

#### 해결 2 - 추상 클래스의 하위 클래스 사용하기

추상 클래스의 하위 클래스에서는 ```equals``` 규약을 지키면서도 값을 추가할 수 있다.    

상위 클래스의 인스턴스를 직접 만드는 게 불가능하기 때문에, 하위 클래스끼리의 비교가 가능하다.

**✔️ 일관성(consistency)**  

> 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다.  

- 가변 객체의 경우 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있다.  

- 불변 객체는 한번 다르면 끝까지 달라야 한다. 

- 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.  

ex) ```java.net.URL```의 ```equals```는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교하는데, 
호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하므로 그 결과가 항상 같다고 보장할 수 없다.  

→ ```equals```는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 한다.

**☑️ null-아님**  

모든 객체가 null과 같지 않아야 한다.  

**잘못된 명시적 ```null``` 검사**

```java
@Override
public boolean equals(Object o) {
  if(o == null) { 
      return false;
  }
}
```

**올바른 묵시적 ```null``` 검사 - 이쪽이 낫다.**

``` java
@Override
public boolean equals(Object o) {
  if(!(o instanceof MyType)) { 
      return false;
  }
  MyType myType = (MyType) o;
}
```  

동시성을 검사하려면 ```equals```는 건네받은 객체를 적절히 형변환한 후 필수 필드들의 값을 알아내야 한다.  

따라서, 형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야 한다.  
입력이 null이면 타입 확인 단계에서 false를 반환하므로 null 검사를 명시적으로 하지 않아도 된다.

##

#### ✔️ 정리: 양질의 equals 메서드 구현 방법 

1. **```==```연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.**   

자기 자신이면 ```true```를 반환한다. 단순한 성능 최적화용으로 비교 작업이 복잡한 상황일 때 값어치를 한다.  

2. **instanceof 연산자로 입력이 올바른 타입인지 확인한다.**   

가끔 해당 클래스가 구현한 특정 인터페이스를 비교할 수도 있다.  

이런 인터페이스를 구현한 클래스라면 ```equals```에서 (클래스가 아닌) 해당 인터페이스를 사용해야한다.  

ex) Set, List, Map, Map.Entry 등 컬렉션 인터페이스들  


**3. 입력을 올바른 타입으로 형변환 한다.**  

2번에서 ```instanceof``` 연산자로 입력이 올바른 타입인지 검사 했기 때문에 이 단계는 100% 성공한다.  
입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.  
모두 일치해야 ```true```를 반환한다.   

##

**✔️ equals 구현 시 주의할 추가 사항**   

**- 기본 타입**  : ```==``` 연산자 비교   

**- 참조 타입** : ```equals``` 메서드로 비교   

**- float, double 필드**  

: 정적 메서드 ```Float.compare(float, float)```와 ```Double.compare(double, double)```로 비교
```Float.equals(float)```나 ```Double.equals(double)```은 오토 박싱을 수반해 성능상 좋지 않다.  

**- 배열 필드**
: 원소 각각을 지침대로 비교한다. 모두가 핵심 필드라면 ```Arrays.equals()```를 사용한다.   

**- null 정상값 취급 방지**   
: ```Object.equals(object, object)```로 비교하여 ```NullPointException``` 발생을 예방한다.  

**- 비교하기 복잡한 필드를 가진 클래스**  
: 필드의 표준형(canonical form)을 저장한 후 표준형끼리 비교   

**- 필드의 비교 순서는 ```equals``` 성능을 좌우한다.**  

: 다를 가능성이 크거나 비교하는 비용이 싼 필드부터 비교   
파생 필드가 객체 전체 상태를 대표하는 경우, 파생 필드부터 비교 

**- ```equals```를 재정의할 땐 ```hashCode```도 반드시 재정의하자**  
너무 복잡하게 해결하려 들지 말자.   
```Object``` 외의 타입을 매개변수로 받는 ```equals``` 메서드는 선언하지 말자.   
```public boolean equals(MyClass o)```: 입력 타입이 ```Object```가 아니므로 오버로딩한 것이다.  


##### 잘 구현된 예

``` java
public class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if(val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if(o == this) {
            return true;
        }

        if(!(o instanceof PhoneNumber)) {
            return false;
        }

        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}
```
 ##

#### ☑️ AutoValue 프레임워크
```equals```(```hashCode```도 마찬가지)를 작성하고 테스트하는 작업을 대신해줄 오픈 소스.  

클래스에 애너테이션 하나만 추가하면 AutoValue가 이 메서드들을 알아서 작성해준다.


#### ☑️ 결론  

**꼭 필요한 경우가 아니면 ```equals```를 재정의하지 말자.**    

많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다.    

재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.
