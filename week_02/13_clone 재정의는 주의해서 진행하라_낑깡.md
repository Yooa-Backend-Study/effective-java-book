# 아이템13 | clone 재정의는 주의해서 진행하라 

1. clone 메서드를 잘 동작하게끔 해주는 구현 방법
2. 언제 그렇게 해야 되는지?
3. 가능한 다른 선택지

---

객체를 복사하기 위해서는 Cloneable을 구현해주고, clone 메서드를 재정의해야 한다. 


**clone()의 사용 문제점** 
1. Cloneable 안에 존재하지도 않는다.
2. 그래서 Object 클래스에 있는 clone 메서드를 override해서 사용하게 되는데, 접근자가 심지어 protected이다.
=> Cloneable을 구현했다고 해서 외부 객체에서 clone 메서드를 호출 할 수 없다.

<img width="465" alt="13_1" src="https://github.com/lorlorv/algorithm_JAVA/assets/78305392/cc71e5aa-9154-4102-8755-d30d00281313">

**그럼 Cloneable을 왜 구현해야 하는데?** 

clone 하는 객체의 필드들을 하나하나 복사한 객체를 반환해주며 그렇지 않은 클래스의 인스턴스에서 호출한다면 CloneNotSupportedException을 던지는 clone 메서드의 동작 방식을 결정해준다.

즉, Cloneable 인터페이스를 구현하는 클래스만이 clone 메서드를 호출할 수 있으며, 이 인터페이스를 구현하지 않은 클래스는 clone 메서드를 호출하면 CloneNotSupportedException 예외가 발생한다.




### Clone 메서드의 허술한 일반 규약
<img width="650" alt="13_2" src="https://github.com/lorlorv/algorithm_JAVA/assets/78305392/e87f53f5-9e4d-493a-b504-cfa0784be0f6">

### 왜 허술하다고 하는 걸까?

**1. 재정의 된 clone 메서드의 문제** 

위의 규약은 강제성이 없다는 점만 빼면 생성자 연쇄와 살짝 비슷한 매커니즘이다.
   
    - 객체 생성 및 초기화 
    - 코드 재사용 및 중복 최소화
    - 객체 복제 및 독립성 

그렇다면 clone 메서드가 super.clone()이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해볼까?

```java
package item13;

// 생성자 사용해서 clone
public class Parent implements Cloneable {
    String name;

    public Parent(String name) {
        this.name = name;
    }

    // clone 메서드 오버라이딩
    @Override
    public Parent clone() {
        // 생성자를 호출해 얻은 인스턴스를 반환
        return new Parent(this.name); 
    }

    public void printInfo() {
        System.out.println("Name: " + name);
    }

    public static void main(String[] args) {
        Parent original = new Parent("parent");

        // clone 메서드를 호출하여 객체 복제
        Parent copy = original.clone();

        // 복제된 객체 출력
        copy.printInfo();
        System.out.println("[parent clone] : " + copy);

    }
}
```
> [parent origin] : item13.Parent@71be98f5
> 
> Name: parent
> 
> [parent clone] : item13.Parent@6fadae5d
> 
> Name: parent


하지만 이 클래스의 하위 클래스에서 super.clone을 호출한다면?
```java
package item13;

public class Child extends Parent {
    int child;

    public Child(String name, int child) {
        super(name);
        this.child = child;
    }

    @Override
    public Child clone() {
        return (Child) super.clone();
    }

    public static void main(String[] args) {
        // clone 메서드를 호출하여 객체 복제
        Child original = new Child("child", 1);
        System.out.println("[child origin] : " + original);

        Child copy = original.clone();

        // 복제된 객체 출력
        copy.printInfo();
        System.out.println("[child clone] : " + copy);
        copy.printInfo();
    }
}
```
> Exception in thread "main" java.lang.ClassCastException: class item13.Parent cannot be cast to class item13.Child (item13.Parent and item13.Child are in unnamed module of loader 'app')
at item13.Child.clone(Child.java:13)
at item13.Child.main(Child.java:19)

`클래스 B가 클래스 A를 상속할 때, 하위 클래스인 B의 clone은 B 타입의 객체를 반환해야 한다. 그런데 A의 clone이 자신의 생성자, 즉 new A(…) 로 생성한 객체를 반환한다면, B의 clone도 A 타입 객체를 반환할 수밖에 없다. 달리 말해 super.clone을 연쇄적으로 호출하도록 구현해두면 clone이 처음 호출된 상위 클래스의 객체가 만들어진다.`

즉, super.clone()으로 얻어올 수 있는 객체가 child가 아니라 parent이므로 다운 캐스팅에 실패한다. 

clone 메서드의 반환타입은 복제 대상 객체의 원래 클래스와 동일해야한다는 일반 규약이 있지만, 이 상황에서는 parent를 반환하기 때문에 에러가 발생한다.

<br>

**2. final 필드 문제** 

**가변 상태를 참조하지 않는 클래스일 때**,
클래스에 정의된 모든 필드는 원본 필드와 똑같은 값을 가지므로 더 손볼 것이 없다.

→ 쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 clone 메소드를 제공하지 않는게 좋다.

만약, 정의하게 된다면 
```java 
// 코드 13-1 가변 상태를 참조하지 않는 클래스용 clone 메서드 (79쪽)
    @Override public PhoneNumber clone() throws CloneNotSupportedException {
        return (PhoneNumber) super.clone();
    }
```
→ super.clone이 Object를 반환하지만, PhoneNumber를 반환하게 만들어 클라이언트가 형변환하지 않게끔 해준다.

⇒ 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.

### 하지만 가변 상태를 참조한다면 ? 재앙 is coming ...

```java
package item13;

public class User implements Cloneable{
    private String name;
    private Info info;

    public User(String name, Info info) {
        this.name = name;
        this.info = info;
    }

    @Override
    public User clone()  {
        try {
            return(User) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    public static void main(String[] args) {
        User original = new User("eunseo", new Info(18));
        System.out.println("original age : " + original.getInfo().getAge());

        User copied = original.clone();

        Info info = copied.getInfo();
        
        info.setAge(24);

        System.out.println("After copy ... ");

        System.out.println("original age : " + original.getInfo().getAge());
        System.out.println("copied age : " + copied.getInfo().getAge());
    }
}
```
> original age : 18
> 
> After copy ...
>
> original age : 24
> 
> copied age : 24

clone 메서드가 단순히 super.clone의 결과를 그대로 반환했을 때, 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다.

왜? 

위의 기본 규약을 보면

<img width="685" alt="13_3" src="https://github.com/lorlorv/algorithm_JAVA/assets/78305392/a96f95c0-5063-4d8e-9415-f8d5868490fa">

> **객체의 `clone` 메서드 동작**
> 
>
> - 이 메서드는 현재 객체와 동일한 클래스의 새로운 인스턴스를 생성한다.
> - 새로운 인스턴스는 현재 객체의 필드를 정확히 동일한 내용으로 초기화한다. 이 작업은 필드 간의 할당 연산을 통해 이루어진다.
> - 필드 내용 자체는 복제되지 않고, 현재 객체와 동일한 참조를 가진다. 이것은 **"얕은 복사(shallow copy)"** 작업을 수행함을 의미한다. 즉, 복제된 객체의 필드는 원본 객체와 동일한 객체를 참조한다.

이기 때문이다.

⇒ 즉, Clone 메소드는 사실상 생성자와 같은 효과를 낸다. clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

→ Info라는 객체를 똑같이 복사해서 만들어줘야 함

```java
package item13;

import java.util.Hashtable;
import java.util.Map;

public class User implements Cloneable{
    private String name;
    private Info info;

    public User(String name, Info info) {
        this.name = name;
        this.info = info;
    }

    @Override
    public User clone()  {
        try {
            User cloned = (User) super.clone();
            cloned.info = (Info) info.clone();
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    public static void main(String[] args) {
        User original = new User("eunseo", new Info(18));
        System.out.println("original age : " + original.info.getAge());

        User copied = original.clone();

        Info info = copied.info;

        info.setAge(24);

        System.out.println("After copy ... ");

        System.out.println("original age : " + original.info.getAge());
        System.out.println("copied age : " + copied.info.getAge());
    }
}


```

>original age : 18
> 
>After copy ...
> 
> original age : 18
> 
>copied age : 24

**하지만, 만약 info 필드가 final이었다면, final 필드에는 새로운 값을 할당할 수 없기 때문에 위의 방식이 작동하지 않는다.**

⇒ 직렬화와 마찬가지로 Cloneable  아키텍처는 ‘가변 객체를 참조하는 필드는 fianl로 선언하라’  는 일반 용법과 충돌한다. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있음 

### 만약, 위의 예시 처럼 clone을 재귀적으로 사용하지 못하는 경우가 있다면?
> Hash Table 
> 해시 테이블의 복제본은 자신만의 버킷 배열을 갖긴 하지만, 이 배열은 원본과 같은 연결 리스트를 참조한다. 
> 
> => 각 버킷을 구성하는 연결리스트까지도 복사가 되어야 함
> 
> => 이럴 때 사용해야하는 것이 깊은 복사이다.

HashTable.Entry는 깊은 복사를 지원하도록 보강되었고, 적절한 크기의 새로운 버킷 배열을 할당한 다음, 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은 복사를 수행한다.

(스택 할당량 문제로 반복자 사용하여 수정 필요)

---

### 생성자 내에서는 재정의 될 수 있는 메소드를 호출하지 말자

만약, clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복사 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다.

### ClassNotSupportedException

재정의한 clone 메서드에서는 throws 절을 없애야 clone 메소드를 호출할 때마다 예외처리를 하지 않아도 된다.

### 하위 클래스에서 Clone을 지원하지 못하게 하는 방법
상속용 클래스는 Cloneable을 구현해서는 안되기 때문에 제대로 작동하는 clone 메서드를 구현해 protected로 두고 CloneNotSupportedException도 던질 수 있다고 선언하는 것이다.

**아예 clone 메서드를 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수도 있다.**

### 스레드 안정성을 고려한다면 적절히 동기화해야 한다.
```java 
@Override
public synchronized Object clone() {
  try {
    Object result = super.clone();
  } catch(CloneNotSupportedException e) {
  }
}
```

---

### 복사 생성자와 복사 팩터리 
```java 
public Child(Child child) { // 복사 생성자
    this.childField = child.getChildField();
}

// 복사 생성자 내의 레퍼런스들도 복사 생성자의 체인을 통해서 복사해주어야함
public Parent {
	int parentField;
	Child child;
	public Parent(Parent p) {
		this.parentField = p.getChildField();
		this.child = new Child(p.child);
	}
}

public static Parent newInstance(Parent parent) { // 복사 팩터리
    return new Parent(parent.getParentField());
}
```
1. 언어 모순적이고 위험천만한 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지 않는다. (super.clone())
2. 엉성하게 문서화된 규약 (clone 규약) 에 기대지 않는다.
3. 정상적인 final 필드 용법과 충돌하지 않음
4. 불필요한 check exception 처리가 필요 없음
5. 형변환 필요 없음
6. 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
    ->  원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.

---



### 결론
Cloneable을 이미 구현한 클래스를 확장하는 경우가 아니라면 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공하는 것이 좋겠다.

- 직렬화 방법
