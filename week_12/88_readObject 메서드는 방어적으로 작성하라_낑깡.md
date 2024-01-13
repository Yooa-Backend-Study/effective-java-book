# 아이템 88 | readObject 메서드는 방어적으로 작성하라

## _기본 직렬화 방식을 사용하기로 결정했더라도 readObject 메서드는 제공하자_
객체의 물리적 표현과 논리적 표현이 일치하고, 상황에 따라 값이 변화되어 불변식이 깨지는 객체도 아니라면

**왜?** 

### readObject 메서드는 실질적으로 또 다른 public 생성자이기 때문
```java 
        serializedMember = Base64.getDecoder().decode(base64Member);
        try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember)) {
            try (ObjectInputStream ois = new ObjectInputStream(bais)) {
                // 역직렬화된 Member 객체를 읽어온다.
                Object objectMember = ois.readObject();
```
> 매개변수로 바이트 스트림을 받는 생성자라고 할 수 있다.

그래서 보통의 생성자 처럼
1. 인수가 유효한지 검사
2. 필요하다면 매개변수를 방어적으로 복사해야 함

**왜?** 

**만약, 악의적인 의도로 생성한 임의 바이트 스트림을 readObject 메서드에게 건넨다면 심각한 문제가 생길 수 있기 때문**
### 1. 스트림 조작
```java 
byte[] badSerializedMember = {(byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06....};
try (ByteArrayInputStream bais = new ByteArrayInputStream(badSerializedMember)) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {
        Object objectMember = ois.readObject();
```

→ 클래스의 `불변식`을 망가뜨릴 수 있다.

### 첫 번째 해결방법, 역직렬화된 객체가 유효한지 검사하기
- 실패 시 `InvalidObjectException` 던지게 하여 잘못된 역직렬화 막기
```java
ois.defaultReadObject();
if (start.compareTo(end) > 0) {
    throw new InvalidObjectException(...);
}
```

### 2. 가변 공격
```java
byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
bais.write(ref); // 시작 start 필드 참조 추가
ref[4] = 4; //참조 #4
bais.write(ref); // 종료(end) 필드 참조 추가
```
- `ref` : Java 직렬화 프로토콜의 일부로 해석될 수 있는 특정 바이트 시퀀스
- `0x71` : Java 직렬화 프로토콜에서 참조 타입을 나타내는 데 사용되는 바이트 
- `0x7e` : 참조의 시작을 나타냄
- `5` : 참조되는 객체의 핸들 또는 ID를 나타냄, 이 경우 #5

```java 
// Period 역직렬화 후 Date 참조를 훔친다.
ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bais.toByteArray()));
period = (Period) ois.readObject();
start = (Date) ois.readObject(); //#5
end = (Date) ois.readObject(); //#4
```
정상적으로 생성된 바이트 스트림에 **악의적인 객체 참조**를 추가
- 역직렬화 시 해당 객체에 접근이 가능하게 되어 불변식은 유지한 채 의도적으로 내부의 값을 수정할 수 있음
  ```java
    end.setYear(..) ;
   ```

**인스턴스가 불변이라고 단언했던 클래스일 경우 원인도 모른채 심각한 보안 문제가 일어날 수 있음**

### 두 번째 해결방법, readObject 메서드에서 모든 필드를 반드시 방어적으로 복사하기
**즉, 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.**
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
- 원본 `Date` 객체의 상태를 새로운 `Date` 객체로 복사하면서 원본 객체가 가진 값은 그대로 유지하면서 독립된 인스턴스를 생성
- 불변식 검사 시점과 객체를 사용하는 시점 사이에 객체의 상태가 (데이터 값) 변경될 수 있음
- 이 경우 불변식 검사는 무의미해질 수 있으며 예상치 못한 동작이나 오류 발생 가능 

→ 1. 방어적 복사하여 값은 그대로 유지하되, 외부의 영향을 받지않게끔 가변 객체를 **방어적으로 복사**한다.

→ 2. 방어적 복사로 탄생한 독립적 인스턴스로 **불변식 검사**를 진행한다.

→ 불변식 검사에 성공하였다면, 이미 방어적 복사를 통해 외부와의 연결을 끊었으므로 안전하게 사용 가능

## 기본 readObject 쓸까? 말까?
### transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
😎 `NO` : **커스텀 readObject** 생성 후 `방어적 복사 + 유효성 검사` 진행 


### 결론
악의적인 공격으로 가변 필드가 생성되어 인스턴스가 불변임을 보장하지 못할 수 있으니 기본 직렬화를 선택하였더라도, readObject 메서드를 재정의 하여 방어적 복사와 함께 유효성 검사를 필수로 하자 