# 아이템 89) 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

```java
public class SingletonPerson implements Serializable {
    public static final SingletonPerson INSTANCE = new SingletonPerson(1, "name", 100);

    private SingletonPerson(Integer id, String name, Integer height){
        this.id = id;
        this.name = name;
        this.height = height;
    }

    private final transient Integer id;
    private final String name;
    private final Integer height;
}
```

### 싱글턴 패턴에 직렬화를 적용하면?

- 더 이상 싱글턴이 아니게 된다
- 기본 직렬화를 쓰지 않고, 명시적인 readObject() 를 제공하더라도 싱글톤 객체와는 별개인 (즉, 객체 해시코드 값이 다른) 인스턴스를 반환하게 됨

### **해결 방법 - readResolve 메서드 이용하기**

- **역직렬화 중에 생성된 객체를 다른 객체로 대체**하는 데 사용되는 `readResolve` 메서드를 통해, 기존 객체의 참조를 반환하면 됨. 
→ 역직렬화 과정에서 자동으로 호출되는 `readObject` 메서드가 있더라도 
`readResolve` 메서드에서 반환한 인스턴스로 대체되기 때문에, 
`readObject` 메서드를 통해 만들어진 인스턴스는 더이상 유지되지 않아 **GC 대상이 되어 사라져 싱글톤 보장 가능!**
- 결론은 어차피 역직렬화해서 만들어진 객체 안쓰고 기존 싱글톤 객체 반환할 거니까 `readResolve` 를 **인스턴스 통제 목적으로 사용한다면, 필드는 모두 transient 로 선언해야 한다**

> **readResolve 메서드 호출 시점**
> 
> 1. **`ObjectInputStream`** 이 바이트 스트림에서 객체를 역직렬화함 
> 2. 역직렬화 시 **`readObject`** 메서드가 호출되면서, 해당 객체의 **`readResolve`** 메서드를 찾아 호출
> 3. **`readResolve`** 메서드의 반환값이 실제로 역직렬화된 값으로 반환할 객체로 사용됨

```java
private final Object readObject(Class<?> type)
        throws IOException, ClassNotFoundException
    {
       ... 
       Object obj = readObject0(type, false);
```

```java
private Object readObject0(Class<?> type, boolean unshared) throws IOException {
        ... 
        try {
            switch (tc) {
                ...
                case TC_STRING:
                case TC_LONGSTRING:
                    return checkResolve(readString(unshared));

                case TC_ARRAY:
                    if (type == String.class) {
                        throw new ClassCastException("Cannot cast an array to java.lang.String");
                    }
                    return checkResolve(readArray(unshared));

                case TC_ENUM:
                    if (type == String.class) {
                        throw new ClassCastException("Cannot cast an enum to java.lang.String");
                    }
                    return checkResolve(readEnum(unshared));

                case TC_OBJECT:
                    if (type == String.class) {
                        throw new ClassCastException("Cannot cast an object to java.lang.String");
                    }
                    return checkResolve(readOrdinaryObject(unshared));
								...
}
```

```java
private boolean enableResolve; // true 인 경우 resolveObject() 호출

private Object checkResolve(Object obj) throws IOException {
	// resolveObject 메서드가 활성화되었고, 주어진 객체에 예외가 연결되어 있지 않은 경우
        if (!enableResolve || handles.lookupException(passHandle) != null) {
            return obj;
        }
	// resolveObject를 호출 -> obj 그대로 반환하는 메서드
        Object rep = resolveObject(obj);
        if (rep != obj) {
            if (rep != null) {
                if (rep.getClass().isArray()) {
                    filterCheck(rep.getClass(), Array.getLength(rep));
                } else {
                    filterCheck(rep.getClass(), -1);
                }
            }
	    // 객체를 대체하고 핸들 테이블을 업데이트
            handles.setObject(passHandle, rep);
        }
	// 주어진 객체 그대로 반환
        return rep;
    }

/*
    ObjectInputStream의 신뢰받는 하위 클래스가 역직렬화 중에 객체를 다른 객체로 대체할 수 있게 해줌
	참고) enableResolveObject 메서드는 객체를 대체할 수 있는 스트림이 신뢰할 수 있는지를 확인
*/
protected Object resolveObject(Object obj) throws IOException {
        return obj;
    }
```

- resolve 메서드를 사용해도 순간 만들어진 역직렬화된 인스턴스에 접근하지 못하게 하려면 추가 작업들이 필요 / 보안에도 취약할 수 있음

### 본질적인 해결 방법 - Enum 타입 활용하기

- 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다
- 예시 코드
    
    ```java
    public enum Elvis {
        INSTANCE;
    
        private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
    
        public void printFavorites() {
            System.out.println(Arrays.toString(favoriteSongs));
        }
    }
    ```
    
- 컴파일 타임에 어떤 인스턴스들이 있는지 알 수 없는 상황이면 열거타입으로 표현하는 것이 불가능하여 readResolve를 사용해야한다

### readResolve를 사용해야 할 경우 주의해야할 점

- final 클래스라면 readResolve 메서드는 private 이어야 한다
    - private으로 선언된 **`readResolve`** 메서드는 자식 클래스에서 직접 접근할 수 없으므로, **final 클래스의 불변성 유지 가능**
- protected나 public 이며 하위 클래스에서 재정의하지 않은 경우, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException 을 일으킬 수 있다

```java
class Parent implements Serializable {
    protected int value;

    public Parent(int value) {
        this.value = value;
    }

    protected Object readResolve() throws ObjectStreamException {
        return new Parent(value + 100);
    }
}

class Child extends Parent {
    public Child(int value) {
        super(value);
    }
}

public class SerializationExample {
    public static void main(String[] args) {
        try {
            // Serialize a Child object
            Child child = new Child(42);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(child);
            oos.close();

            // Deserialize the Child object
            byte[] serializedData = baos.toByteArray();
            ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(serializedData));
            Child deserializedChild = (Child) ois.readObject(); // ClassCastException may occur here
            ois.close();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

![Untitled](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F8c65f92d-6f83-471d-8b8b-2101f652bce7%2Fb66246e4-1f0a-4e20-b204-c8e9bb7b8ffd%2FUntitled.png?table=block&id=04bdc4e7-9cd6-4c4c-9675-122c709eebc2&spaceId=8c65f92d-6f83-471d-8b8b-2101f652bce7&width=2000&userId=edab0db8-cc3f-4181-b4ba-153fc87185e3&cache=v2)

## 정리

- 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 열거 타입을 사용하기
- 열거 타입이 안되면 readResolve 메서드를 작성해서 넣고 모든 참조 타입 인스턴스 필드를 transient로 선언해야 한다.