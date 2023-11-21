# 멤버 클래스는 되도록 static으로 만들라

## ***#멤버 클래스***

- 정적 멤버 클래스
    - 바깥 클래스와 함께 쓰일 때 유용한 public 클래스
- 비정적 멤버 클래스
    - 바깥 클래스의 인스턴스와 **암묵적으로 연결**
    - 어댑터 정의 시 자주 쓰임
    - 멤버 클래스에서 바깥 클래스를 참조할 필요가 없다면 무조건 정적 멤버 클래스로 만들어라
- 익명 클래스
    - 바깥 클래스의 멤버는 아니며, 쓰이는 시점과 동시에 인스턴스가 만들어짐
    - 비정적인 문맥에서 사용될 때만 바깥 클래스 인스턴스 참조 가능
    - 람다 지원 전에 즉석에서 함수 객체를 만들거나 할떄 사용
    - 정적 팩터리 메서드 만들때도 사용
- 지역 클래스

---

```java
public class OuterClass {
	private static int num = 10;

	// 정적 멤버 클래스
	static private class InnerClass {
		void doSomeThing() {
			System.out.println(num);
		}
	}

	public static void main(String[] args) {
		InnerClass innerClass = new InnerClass();
		innerClass.doSomeThing();
	}
}
```

정적 멤버 클래스

- 바깥 클래스의 static 필드에 접근 가능
- **바깥 클래스의 instance 필요 없음 ( 독립적 )**

비 정적 멤버 클래스를 지양해야 하는 이유

- 바깥 클래스 인스턴스 필요
- 필요 없는데 참조해야함.
- GC에서 사라지지도 않음

---

## #***비 정적 멤버 클래스가 필요한 경우***

어댑터 패턴(Adapter Pattern)

기존 코드를 클라이언트가 사용하는 인터페이스의 구현체로 바꿔주는 패턴

- 클라이언트가 사용하는 인터페이스를 따르지 않는 기존 코드를 재사용할 수 있게 해준다.

```java
public class MySet<E> extends AbstractSet<E> {
	@Override
	public Iterator<E> iterator() {
		return new MyIterator();
	}

	@Override
	public int size() {
		return 0;
	}

	private Class MyIterator implements Iterator<E> {
		@Override
		public boolean hasNext() {
			return false;	
		}

		@Override
		public E next() {
			return null;
		}
	}
}
```