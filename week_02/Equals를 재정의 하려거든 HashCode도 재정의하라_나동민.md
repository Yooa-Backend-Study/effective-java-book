# Equals를 재정의 하려거든 HashCode도 재정의하라

- Equals에서 사용하는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 Hashcode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
- Equals가 두 객체를 같다고 판별했다면, Hashcode는 똑같은 값을 반환해야 한다
- Equals가 두 객체를 다르다고 판별했더라도, Hashcode는 다를수도 있다.

---

***Hash 자료구조의 비교 순서***

![Untitled](Equals%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8C%E1%85%A2%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8B%E1%85%B4%20%E1%84%92%E1%85%A1%E1%84%85%E1%85%A7%E1%84%80%E1%85%A5%E1%84%83%E1%85%B3%E1%86%AB%20HashCode%E1%84%83%E1%85%A9%20%E1%84%8C%E1%85%A2%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8B%E1%85%B4%E1%84%92%E1%85%A1%E1%84%85%E1%85%A1%20d9d6a10b94db47b098cf80fa08aaddb4/Untitled.png)

- 동작원리에서 볼 수 있듯, 해시 자료구조는 해시코드를 먼저 비교하게 되는데 만약 해시코드가 다르다면 Equals는 실행도 되지 않고 다른 객체로 인식하게 된다.

---

***Hash Collision***

우리는 보통 Hash자료구조는 해시 버킷을 가지고있고, 그 속에 데이터가 담기게 되는데 해시값이 같은 객체가 삽입될 때 발생하는 문제를 Hash Collision이라고 부릅니다.

여기서 위의 동작 원리를 적용해보면, 다음과 같은 로직을 따르게 됩니다. ( HashMap이라고 가정 )

- 객체 A, B가 존재할 때, A, B는 같은 HashCode를 반환한다고 가정
    - A와 B는 Equals는 다르지만, HashCode가 같은 경우입니다.
- A가 먼저 삽입되고, B가 삽입. 이 때 Hash Collision이 발생
- Java의 HashMap은 Separate Chaining을 따르니 LinkedList에 B가 담기게 됩니다.

위의 순서를 조금 정리를 해 보면

- 데이터가 Hash 자료구조에 처음 담기게 될 때는 HashCode를 기반으로 담기고,
- 충돌이 일어났을 경우에는 Equals로 비교를 하기 시작합니다.
- 이중 해싱은 다른 경우입니다.

---

***Equals와 HashCode를 각각 재정의 해야 하는 이유***

*둘 다 오버라이딩 안한 경우*

필드들이 모두 같은 경우에도 Heap주소가 다르기 떄문에 다르다는 판정을 내리게 됩니다.

*Equals만 오버라이딩 한 경우*

해시 자료구조의 동작 원리에 따라 A, B객체의 필드 값들이 같음에도 불구하고 Hash에서 다르다고 판정을 내릴 수도 있습니다.

```java
public class TestClass{

	static class Node {
		private int h;
		private int w;

		public Node(int h, int w) {
			this.h = h;
			this.w = w;
		}

		@Override
		public boolean equals(Object o) {
			if (this == o)
				return true;
			if (o == null || getClass() != o.getClass())
				return false;
			Node node = (Node)o;
			return h == node.h && w == node.w;
		}
	}

	public static void main(String[] args) {
		Node node1 = new Node(1,2);
		Node node2 = new Node(1,2);

		Map<Node, Integer> hm = new HashMap<>();
		hm.put(node1, 1);
		hm.put(node2, 1);

		System.out.println(hm.size()); // 2
	}
}
```

HashCode만 오버라이딩 한 경우

Hash 자료구조 동작 원리에 따라, 값이 같은 두 객체가 들어와도 HashCode까지는 잘 판별하지만 Equals가 재정의 되어있지 않아 Object의 Equals를 사용하기 떄문에 둘이 다른 객체로 판별하게 된다. 때문에 두 객체를 해시 자료구조에 담아도 사이즈는 2가 나온다.

 

```java
public class TestClass {

	static class Node {
		private int h;
		private int w;

		public Node(int h, int w) {
			this.h = h;
			this.w = w;
		}

		@Override
		public int hashCode() {
			return Objects.hash(h, w);
		}
	}

	public static void main(String[] args) {
		Node node1 = new Node(1,2);
		Node node2 = new Node(1,2);

		System.out.println(node1.hashCode() == node2.hashCode());

		Map<Node, Integer> hm = new HashMap<>();
		hm.put(node1, 1);
		hm.put(node2, 1);

		System.out.println(hm.size()); // 2
	}
}
```

그러면 여기서 Node(1, 2)로 값을 꺼냈을 때 결과는 어떻게 될까 ?

```java
public static void main(String[] args) {
		Node node1 = new Node(1,2);
		Node node2 = new Node(1,2);

		System.out.println(node1.hashCode() == node2.hashCode());

		Map<Node, Integer> hm = new HashMap<>();
		hm.put(node1, 1);
		hm.put(node2, 1);

		Integer result = hm.get(new Node(1, 2));
		System.out.println(result); // what ??

		System.out.println(hm.size()); // 2
	}
```

---

***Equals와 HashCode를 만들 때***

- Equals 정의 시 비교에 사용되지 않는 필드는 반드시 제외하라

굳이 31인 이유 ?

- 홀수이면서 소수인 수. 짝수이면서 오버플로우가 발생한다면 시프트연산과 같은 효과를 내게 되는데, 이 경우 정보를 잃을 수 있다. 소수를 곱하는 이유는 명확하지 않지만 일종의 규약이다. 결과적으로 큰 이유가 없다면 31을 사용하자
    - 그냥 인텔리제이에서 만들어주는 것을 사용해도 되고, 31이 아니어도 다른 숫자를 사용하는 기업도 간혹 있다.

해시코드를 직접 정의한 경우 해시코드를 계산하는데 비용이 많이 든다면 ?

- 해시코드를 캐싱하는 방법이 있다. 단, 이 방법은 불변클래스의 경우만 가능하다.
- 또 하나는 Lazy Initialization 방법이 있다. 해시코드가 사용되는 첫 경우에만 동작시키는 방법이다.