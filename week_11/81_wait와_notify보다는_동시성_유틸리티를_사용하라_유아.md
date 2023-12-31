# wait()과 notify()

Java에는 동시성을 다루기 위해서 wait와 notify를 지원한다. 다만, 이 메서드들은 Java 5부터 지원하는 다양한 동시성 유틸리티 덕분에 wait와 notify를 사용할 이유가 줄었다.

synchronized 키워드를 이용해서 동기화를 하면, 한번에 한 스레드만 임계영역에 들어갈 수 있으므로 프로그램 효율이 떨어진다.

> 실제 synchronized 키워드가 있는 메서드와 없는 메서드의 시간 차이는 ns 기준 1000배 차이가 난다.

이런 효율을 높이기 위해 만들어낸 것이 wait()과 notify()이다.

Object 클래스에 정의되어 있으며, 동기화 블록 내에서만 사용할 수 있다.

- wait() : 객체의 lock을 풀고 스레드를 해당 객체의 waiting pool에 넣는다.
- notify() : waiting pool에 대기중인 스레드 중의 하나를 깨운다.
- notifyAll() : waiting pool에서 대기중인 모든 스레드를 깨운다.
  - 일반적으로 notifyAll()이 합리적이고 안전하다. 꺠어나야 하지만 깨어나지 못한 스레드들을 깨울 수 있고, 혹여나 조건이 만족되지 않으면 어차피 다시 대기할테니까.

표준 예시 하나를 살펴보자.

```java
class Account {
    int balance = 1000;

    public synchronized void withdraw(int money) {
        while(balance < money) {
            try {
                wait(); // 스레드가 락을 풀고 웨이팅 풀에 넣어 기다린다.
            } catch(InterruptedException e) { }
        }
        balance -= money;
    }

    public synchronized void deposit(int money) {
        balance += money;
        notify(); // 통지
    }
}
```

스레드가 락을 풀고 대기함으로써, 입금하려는 다른 스레드가 락을 얻어 동작을 수행할 수 있다.
잔고가 충분하면 출금이 진행될 것이고, 잔고가 부족하면 wait()을 해서 waiting pool을 갈 것.

이처럼 while과 같은 루프로 감싸서 wait시키는 것이 관용적이라 대부분의 레거시 동시성 코드는 저렇게 구현된다고 한다.

# 동시성 유틸리티

java.util.concurrent 패키지가 제공하는 고수준 동시성 유틸리티는 다음과 같은 범주로 나눌 수 있다.

- 실행자 프레임워크 `Executor Framework`
  - ExecutorService, Executors 등을 지원하는 실행자 프레임워크
- 동시성 컬렉션 `Concurrent Collections`
- 동기화 장치 `Synchronizer`

## 동시성 컬렉션

List, Queue, Map 과 같은 표준 컬렉션 프레임워크에 동시성을 가미해 구현한 고성능 컬렉션이다. 높은 동시성을 달성하기 위해서 동기화를 각자의 내부에서 수행한다. 따라서 동시성을 무력화 하는 것은 불가능하고 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

자바의 컬랙션에도 synchronized 키워드를 사용한 자료 구조들이 있다. Vector, Hashtable, Collections.synchronizedXXX() 같은게 대표적이다.

### Collections.synchronizedXXX의 문제점

아까 이야기했듯, synchronized는 오버헤드 문제를 갖고 있다.
이들은 `single lock object`를 사용하여 데이터 구조에 배타적인 접근을 정의하는 방식 사용하며 이를 **대략적인 동기화(coarse-grained synchronization)**라고 한다.

![](https://velog.velcdn.com/images/jmjmjmz732002/post/4bd767b7-d10f-49f8-84a9-29cec796623c/image.png)
위와 같이 6개의 스레드가 있는 경우, 하나의 스레드가 lock을 획득하고 동기화된 컬렉션에 접근할 수 있다. 스레드들이 서로 다른 버킷의 키에 접근하려는 겨웅에도 하나의 스레드만 가능하기에 나머지 스레드들은 lock을 얻기 위해 순차적으로 대기한다.

이런 문제들은 Concurrent Collections으로 해결 할 수 있다.
Concurrent Collections은 synchronized와 달리 `lock striping`같은 좀 더 세분화된 락을 사용하여 여러 쓰레드가 동시 접근을 가능하게 한다.

### lock striping

lock이 여러 버킷 또는 스트라이프(stripe)에서 발생하는 기술이다. 즉, 버킷에 액세스하면 전체 데이터 구조가 아닌 해당 버킷만 잠긴다.

예를 들어, `ConcurrentHashMap`은 기본적으로 16개의 버킷을 가지며, 각 버킷마다 자체적으로 lock을 가지고 있기 때문에 총 16개의 lock을 갖고 있다. 따라서 별도의 버킷에 있는 키에 액세스하는 스레드는 동시에 접근이 가능하다.

그러다보니 clear()와 같이 전체 데이터를 독점적으로 사용해야할 경우, 단일 락을 사용할 때보다 동기화 시키기도 어렵고 자원도 많이 소모하게 된다. 또한, size(), isEmpty()같은 연산이 최신값을 반환하지 못할 수도 있다는 단점이 있다.

종류

- List : CopyOnWriteArrayList
- Map : ConcurrentMap, ConcurrentHashMap
- Set : CopyOnWriteArraySet
- SortedMap : ConcurrentSkipListMap
- SortedSet : ConcurrentSkipListSet
- Queue : ConcurrentLinkedQueue

### CopyOnWrite

컬렉션의 데이터를 변경하는 작업시 내부적으로 복사본을 하나 더 만들어 작업하는 방식이다.

iterate 중에 add/remove 수행하면 비동기화 컬렉션의 경우 ConcurrentModificationException이 발생하고, 동기화 컬렉션의 경우 lock을 걸어 쓰레드가 동시접근 할 수 없다.

CopyOnWrite은 iterate 중에 add/remove 수행하면 동일한 데이터 복사본을 만들어서 add/remove 작업을 한다. iterate는 원본 데이터로 하고 있고 작업이 끝나면 사라지고 add/remove 작업을 한 복사본이 최종적으로 남게 된다.

> 복사본을 만들어 작업하는 만큼 성능이슈가 있을 수 있다. 따라서 크기가 작고, 읽기 작업이 많을 때 사용하기 좋으며 추가, 삭제 작업이 많다면 성능이 떨어진다.

## 동기화 장치

BlockingQueue 는 Queue를 확장한 컬렉션인데 추가된 메서드 중 take는 큐의 첫 원소를 꺼낸다. 이때 만약 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다. 이런 특성 덕분에 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다.

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 해 서로의 작업을 조율할 수 있도록 해준다. 가장 자주 쓰이는 동기화 장치는 CountDownLatch 와 Semaphore다. 그리고 가장 강력한 동기화 장치는 바로 Phaser다.

### CountDownLatch

스레드를 n개 실행했을 때, 일정 개수의 스레드가 모두 끝날 때 까지 기다려야지만 다음으로 진행할 수 있거나 다른 스레드를 실행시킬 수 있는 경우 사용한다.

e.g. 리스트에 어떤 자료구조가 있고, 각 자료구조를 병렬로 처리한 후 batch로 데이터베이스를 업데이트 한다거나 다른 시스템으로 push하는 경우

#### 어떤점이 이를 가능하게 하는가?

CountDownLatch를 초기화할 때 정수값 count를 넣어준다. 스레드는 마지막에서 countDown() 메서드를 불러준다. 그러면 초기화 떄 넣어준 정수값이 하나 내려간다.
즉, 각 스레드는 마지막에서 자신이 실행완료했음을 countDown() 메서드로 알려준다. 이 스레드들이 끝나기를 기다리는 쪽 입장에서는 await() 메서드를 불러준다. 그러면 현재 메서드가 실행중인 메인 스레드는 더 이상 진행하지 않고, CountDownLatch의 count가 0이 될 떄까지 기다린다.

> 0이라는 정수값이 게이트(Latch) 역할을 한다. 카운트다운이 되면 게이트가 열리는 것.

---

참고

- [netjstech - lock stripping in java](https://www.netjstech.com/2016/05/lock-striping-in-java-concurrency.html)
