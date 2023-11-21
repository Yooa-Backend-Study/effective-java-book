# 아이템 37 | ordinal 인덱싱 대신 EnumMap을 사용하라

## ordinal()을 사용해서 값을 꺼낼 때의 문제점
1. 비검사 형변환 수행
```java 
Set<Food>[] foodsByType = (Set<Food>[]) new Set[Food.Type.values().length];
```
2. 출력 결과 커스텀 필요
```java 
for (int i = 0; i < foodsByType.length; i++) {
    System.out.printf("%s : %s%n", Food.Type.values()[i], foodsByType[i]);
}
```
3. 타입 불안전
```java 
for (Food food : basket) {
    foodsByType[food.type.ordinal()].add(food); // KOREAN = 0, CHINESE = 1, JAPANESE = 2
}
```

## 해결책 `EnumMap`
열거 타입을 키로 사용하는 Map 구현체

내부에서 배열을 사용 → Map의 타입 안정성 + 배열의 성능 


### 1. EnumMap 특징
1. 타입 안정성
```java 
    /**
     * Creates an empty enum map with the specified key type.
     *
     * @param keyType the class object of the key type for this enum map
     * @throws NullPointerException if <tt>keyType</tt> is null
     */
    public EnumMap(Class<K> keyType) {
        this.keyType = keyType;
        keyUniverse = getKeyUniverse(keyType);
        vals = new Object[keyUniverse.length];
    }
```
```java 
Map<Food.Type, Set<Food>> foodsByType = new EnumMap<>(Food.Type.class);
```
- EnumMap의 생성자가 받는 키 타입의 Class 객체는 `한정적 타입 토큰` → 런타임 제네릭 타입 정보를 제공 → 타입 안정성 !

2. 내부 ordinal 사용 → `O(1)`
```java 
    /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for this key, the old
     * value is replaced.
     *
     * @param key the key with which the specified value is to be associated
     * @param value the value to be associated with the specified key
     *
     * @return the previous value associated with specified key, or
     *     <tt>null</tt> if there was no mapping for key.  (A <tt>null</tt>
     *     return can also indicate that the map previously associated
     *     <tt>null</tt> with the specified key.)
     * @throws NullPointerException if the specified key is null
     */
    public V put(K key, V value) {
        typeCheck(key);

        int index = key.ordinal();
        Object oldValue = vals[index];
        vals[index] = maskNull(value);
        if (oldValue == null)
            size++;
        return unmaskNull(oldValue);
    }
```
```java 
    /**
     * Returns the value to which the specified key is mapped,
     * or {@code null} if this map contains no mapping for the key.
     *
     * <p>More formally, if this map contains a mapping from a key
     * {@code k} to a value {@code v} such that {@code (key == k)},
     * then this method returns {@code v}; otherwise it returns
     * {@code null}.  (There can be at most one such mapping.)
     *
     * <p>A return value of {@code null} does not <i>necessarily</i>
     * indicate that the map contains no mapping for the key; it's also
     * possible that the map explicitly maps the key to {@code null}.
     * The {@link #containsKey containsKey} operation may be used to
     * distinguish these two cases.
     */
    public V get(Object key) {
        return (isValidKey(key) ?
                unmaskNull(vals[((Enum<?>)key).ordinal()]) : null);
    }
```
4. equals 연산 시 전체 순회를 통해 Key, Value 일치 확인
   → 성능 저하 주의 !
5. TreadSafe하지 않아 Collectors.synchronizedMap으로 wrap 권장 

## 코드를 더 줄이기 위한 Stream 사용
### 1. Stream 사용 (EnumMap X)
```java 
System.out.println(Arrays.stream(menu).collect(Collectors.groupingBy(p -> p.type)));
```
```
public static <T, K> Collector<T, ?, Map<K, List<T>>>
    groupingBy(Function<? super T, ? extends K> classifier) {
        return groupingBy(classifier, toList());
}
```
```java 
public static <T, K, A, D> Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                          Collector<? super T, A, D> downstream) {
        return groupingBy(classifier, HashMap::new, downstream);
}
```
Collectors.groupingBy의 일반적인 Map 구현체는 해시맵(HashMap)이나 트리맵(TreeMap) 등이 사용됨
→ 더 많은 공간 차지, 성능 저하 가능

### 2. Stream 사용 (EnumMap O)
```java 
System.out.println(Arrays.stream(menu)
                .collect(Collectors.groupingBy(p -> p.type, () -> new EnumMap<>(Food.Type.class), Collectors.toSet())));
```
```java 
public static <T, K, D, A, M extends Map<K, D>>
    Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                                  Supplier<M> mapFactory,
                                  Collector<? super T, A, D> downstream) {
```
- 매개변수 3개짜리 Collectors.groupingBy method


> `Stream EnumMap(O)` vs `Stream EnumMap(X)`

## 중첩 EnumMap을 사용한 예제 분석

```java 
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
        ...
        
        
private static final Map<Phase, Map<Phase, Transition>> m = Stream.of(values()).collect(
                groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
                        
public static Transition from(Phase from, Phase to) {
    return m.get(from).get(to);
}
``` 
```java 
// Collectors.groupingBy
public static <T, K, D, A, M extends Map<K, D>>
    Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                                  Supplier<M> mapFactory,
                                  Collector<? super T, A, D> downstream) {
```
```java 
// Collectors.toMap
public static <T, K, U, M extends Map<K, U>>
    Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper,
                                BinaryOperator<U> mergeFunction,
                                Supplier<M> mapSupplier) {
        BiConsumer<M, T> accumulator
                = (map, element) -> map.merge(keyMapper.apply(element),
                                              valueMapper.apply(element), mergeFunction);
        return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
    }
```
맵의 맵을 초기화하기 위하여 수집기(java.util.stream.Collector) 2개 사용
1. groupingBy : 전이를 이전 상태를 기준으로 묶는다.
    > `classifier` : 키로 매핑
   >
   >>  
   >> MELT(SOLID, LIQUID) : MELT.from <- Phase 타입의 SOLID를 키로 매핑 
   > 
   > `mapFactory` : 원하는 타입의 맵 만들어주기 위한 공급자
   >> 
   >> () -> new EnumMap<>(Phase.class); 명시적으로 EnumMap 사용
   >> 
   >> => 생성된 Map의 Key는 Phase, 값은 downstream에서 모은 결과로 나타남
   > 
   > `downstream` 
   >> EnumMap의 values()를 사용해 MELT, FREEZE, BOIL ... 등을 Stream으로 생성 
   

2. toMap : 이후 상태를 전이에 대응시키는 EnumMap 생성 
    > EnumMap을 만들기 위하여 toMap 사용
   > 
    > `keyMapper` : Key 지정
   >  
   >> Key를 MELT.to <- Phase 타입 LIQUID를 키로 매핑 
   > 
   > `valueMapper` : Value 지정
   >  
   >> Transition 그 자체를 매핑 
   >>  
   >> → MELT(SOLID, LIQUID) 는 <LIQUID, MELT> 로 매핑됨
   > 
   > `mergeFunction` : 같은 키에 매핑된 값이 2개 이상일 때 사용
   > 
   > `mapSupplier` : 어떤 맵을 만들지 지정 
   > 
    > Key : Phase, Value : Transition 인 EnumMap 생성 
   > 
   > => Map<Phase, Map<Phase, Transition>>
   > 

3. 결과 
```java 
public static void main(String[] args) {
        for (Phase src : Phase.values()) {
            for (Phase dst : Phase.values()) {
                Transition transition = Transition.from(src, dst);
                if (transition != null)
                    System.out.printf("%s에서 %s로 : %s %n", src, dst, transition);
            }
        }
    }
```
- from의 매개변수 값(SOLID)으로 매핑된 <Phase, Transition> (LIQUID, MELT) 를 꺼내고, to (LIQUID) 값으로 매핑된 Transition (MELT) 를 꺼냄
- 즉, SOLID와 매핑된 맵에서 LIQUID와 매핑된 값을 꺼낸다.

## 결론 
1. 타입 안정성
2. 배열의 성능
→ **메모리 효율성**

`성능`과 `동시성` 고려하는 것 잊지말기 

