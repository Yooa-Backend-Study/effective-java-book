# [아이템 16] public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

p. 102~104

```java
public class Rectangle {
    public double width;
    public double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    public double computeArea() {
        return width * height;
    }

    public double computePerimeter() {
        return 2 * (width + height);
    }
}
```

해당 클래스는 데이터 필드에 직접 접근 가능하며 불변성이 보장되지 못한 객체이다.

public 클래스에서 데이터 필드를 직접 노출하면 해당 클래스의 내부 구현이 공개 API로서 영구적으로 노출되어야 한다. 그렇게 하면 내부 표현 방식을 변경하기 어렵고, 다른 클래스가 클래스 내부의 데이터를 잘못 사용하거나 수정할 수 있다.

이러한 이유로 public 클래스에서는 데이터 필드를 `private`으로 선언하고 적절한 getter 및 setter 메서드를 통해 접근하는 것이 좋다.

하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.

```java
public class OuterClass {
    private class NestedClass {
        int data; // 데이터 필드를 private으로 선언
    }

    public void doSomething() {
        NestedClass nested = new NestedClass();
        nested.data = 42; // private 중첩 클래스의 데이터 필드에 접근
    }
}
```

이 방식은 동일한 패키지 내에서만 접근할 수 있으며, 외부 패키지에서는 접근할 수 없다. 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. (getter/setter의 코드가 줄어들었으니까.)

`NestedClass`는 `OuterClass` 내부에서만 사용될 수 있으며 외부에서는 직접 접근할 수 없다.
클래스의 내부 표현 방식을 외부로부터 숨기고, `OuterClass`의 변경이 `NestedClass`를 사용하는 클라이언트 코드에 미치는 영향을 제한하는 데 도움을 준다.

> java.awt 패키지의 Point와 Dimension 클래스는 public 클래스의 필드 직접 노출의 나쁜 예시. 외부에서 필드에 자유롭게 접근하고 수정이 가능하여 예기치 않은 동작이나 성능 문제가 발생한다고.

## public 클래스의 불변 필드

public 필드가 불변이더라도 여전히 단점이 크다.
공개적인 API로 기능하는 클래스에서 필드의 표현 방식을 바꾸기란 사실상 어려우며(클라이언트 코드가 필드에 의존하기 있기에.) 이는 코드 확작성이나 유지보수성 측면에서 치명적이다.

또한, 직접 노출된 필드를 읽는 코드는 필드의 값을 읽는 것 이상의 작업을 수행할 방법이 없다. (예를 들어, 필드 값을 읽기 전 유효성 검사를 하거나..)

**캡슐화된 필드를 통해 더 많은 유연성을 얻는 방식**을 택하도록 하자.
