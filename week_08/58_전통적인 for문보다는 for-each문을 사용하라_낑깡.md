# 아이템 58 | 전통적인 for문보다는 for-each문을 사용하라

> 반복자, 인덱스 원소말고 **원소**들이 필요할 경우 for-each문을 사용하자!

## 전통적인 for문을 사용할 경우
1. 변수가 많아질 수록 오류가 생길 가능성이 높아지며, 컴파일 타임에 발견하지 못할 가능성이 높다.
2. 컬렉션이냐 배열이냐에 따라 코드 형태도 달라지기 때문에 주의해야하는 피곤함.
    ```java 
    # 컬렉션
    // ArrayList 생성
        List<String> myList = new ArrayList<>();
        myList.add("Java");
        myList.add("Python");
        myList.add("C++");

       for (int i = 0; i < myList.size(); i++) {
            String element = myList.get(i);
            System.out.println(element);
        }
    ```
    ```java 
    # 배열 
    // 배열 생성
        String[] myArray = {"Java", "Python", "C++"};

        for (int i = 0; i < myArray.length; i++) {
            String element = myArray[i];
            System.out.println(element);
        }
    ```
   
## enhanced `for` statement
: 향상된 `for` 문 
```java 
// 향상된 for문을 이용한 순회
        for (String element : myArray) {
            System.out.println(element);
        }
```

1. 반복자와 인덱스 사용 (X) → 코드 가독성 ⬆️, 오류 가능성 ⬆️
2. 하나의 관용구로 컬렉션과 배열을 모두 처리 가능 
3. 중첩 반복문 순회 시 이점 극대화 

    ```java 
    List<Card> deck = new ArrayList<>();
                
    for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
       for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
          deck.add(new Card(i.next(), j.next()));
    ```
   
   ```java 
    for (Suit suit : suits) {
       for (Rank rank : ranks) {
          deck.add(new Card(suit, rank));
       }
    }    
   ```


## 사용할 수 없는 상황
### 1. 파괴적인 필터링 (destructive filtering)
- 컬렉션 순회하면서 해당 원소를 제거해야 한다면 반복자의 `remove()` 호출 필요 
- 자바 8부터는 Collection의 `removeIf()` 를 사용해 명시적으로 순회하는 일을 피할 수 있다.
    
  ```java 
  # 명시적 순회
   public class IteratorTest {
     public static void main(String[] args) {
         List<Animal> myList = new ArrayList<>();
         myList.add(new Animal("cat"));
         myList.add(new Animal("dog"));
         myList.add(new Animal("bird"));
   
         for (Animal animal : myList) {
             System.out.println(animal.getName());
   
             if (animal.getName().equals("cat"))
                 myList.remove(animal);
         }
     }
  }
   ```
   ![스크린샷 2023-12-09 오후 11 02 56](https://github.com/Yooa-Backend-Study/effective-java-book/assets/78305392/4cb8751a-350f-4c66-8bc1-56139b5b9ddc)
  > 순회하면서 요소를 삭제하기 때문에, 인덱스가 변경되어 일부 요소는 순회하지 않을 수 있다. 
   > 
   > 리스트에서 문제가 발생할 수 있음을 감지하고 `ConcurrentModificationException` 발생시킴
   
   ```java 
  # Iterator의 removeIf 사용
   public static void main(String[] args) {
      List<Animal> myList = new ArrayList<>();
      myList.add(new Animal("cat"));
      myList.add(new Animal("dog"));
      myList.add(new Animal("bird"));


      myList.removeIf(animal -> animal.getName().equals("cat"));
   }
   ```
   ![스크린샷 2023-12-09 오후 11 08 30](https://github.com/Yooa-Backend-Study/effective-java-book/assets/78305392/5c7711ef-e3b7-4eb6-a5ec-3d8e417b8bad)

   - 내부적으로 Iterator를 사용하기 때문에 코드에서 명시적으로 순회를 사용하지 않게 되고, 직접 접근하지 않아 안전
   - `Iterator` 이 컬렉션의 상태를 관리하고 있기 때문 
     1. Fail-Fast 방식
        1. 컬렉션 순회 도중에 컬렉션이 수정되면 즉시 예외를 발생시켜 현재 순회를 중단
        2. 여러 스레드가 동시에 컬렉션을 수정하려는 시도를 감지하고 예외를 던져 안전한 상태를 유지 가능
     2. 내부 메소드 remove()
        1. 삭제하는 요소 위치 기억한 후 위치 다음으로 커서 이동 → next 메서드를 호출했을 때 유효한 요소 반환 가능 
        
### 2. 변형 (transforming)
```java 
        for (Animal a : myList) {
            if (a.getName().equals("dog")){
                myList.set( ? , new Animal("new dog"));
            }
        }
```
- 순회하면서 해당 값을 변경해야할 때 리스트의 반복자나 배열의 인덱스가 필요 !

### 3. 병렬 반복 (parallel iteration)
- 여러 컬렉션을 병렬로 순회해야 한다면? == 중첩 반복문에 대하여 이야기
  - 58-4의 예시
     ```java 
      public class DiceRolls {
      enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }

         public static void main(String[] args) {
             # 필요한 결과
             Collection<Face> faces = EnumSet.allOf(Face.class);
   
             for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
                 for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
                     System.out.println(i.next() + " " + j.next());
   
             System.out.println("***************************");
   
            # 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어할 수 없음
             for (Face f1 : faces)
                 for (Face f2 : faces)
                     System.out.println(f1 + " " + f2);
         }
      }
      ```
  
## 결론
for-each 문은 컬렉션, 배열 뿐만 아니라 Iterable 인터페이스를 구현한 객체라면 사용이 가능하므로 `순회` 를 중점으로 사용하는 타입을 작성해야 한다면 Iterable을 구현하는 쪽으로 생각하자.

---

## 번외 
### for-loop vs Stream 성능 비교
http://www.angelikalanger.com/Conferences/Videos/Conference-Video-GeeCon-2015-Performance-Model-of-Streams-in-Java-8-Angelika-Langer.html

https://sigridjin.medium.com/java-stream-api%EB%8A%94-%EC%99%9C-for-loop%EB%B3%B4%EB%8B%A4-%EB%8A%90%EB%A6%B4%EA%B9%8C-50dec4b9974b
1. `primitive type` : int를 저장하는 배열을 만들고, 배열에서 가장 큰 원소를 찾는 작업
   1. for-loop의 압도적 승리
      1. JIT Compiler가 for-loop를 오랜 시간동안 다뤄왔기 때문에, 내부 최적화 작업이 이미 잘 되어 있는 반면, 스트림은 2015년 이후에 도입되었으므로 아직 최적화 되지 않음을 근거로 들음.
      2. 스트림 파이프라인을 생성하고 종료하는 데 필요한 추가 작업, 중간 연산, 박싱 언박싱에 대한 오버헤드

2. `wrapped type` : ArrayList에 Integer 타입을 저장한다음, Integer 중 가장 큰 원소를 찾는 작업
   1. for-loop가 약 1.27배 정도밖에 빠르지 않음 
      1. Arraylist를 순회하는 비용 자체가 워낙 커서, for-loop와 stream 간의 성능 차이를 압도해버린 것
      2. wrapped type은 primitive type과 달리 stack이 아닌, heap 영역에 저장되기 때문에 간접 참조 방식 
      3. 간접 참조 비용이 직접 참조 비용보다 훨씬 비싸기 때문에 for-loop 컴파일러 최적화 이점이 사라짐
      

3. 만약, 순회 비용보다 `계산 비용`이 훨씬 비싸다면? 
   1. 거의 비슷한 결과값, 더 이상 빠르다고 볼 수 없음 
      1. 함수 내부의 시간 복잡도가 충분히 크다면, stream을 사용했을 때 속도 측면에서 손실이 없다.
