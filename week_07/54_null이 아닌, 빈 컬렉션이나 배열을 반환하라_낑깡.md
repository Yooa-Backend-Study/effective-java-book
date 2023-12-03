# 아이템 54 | null이 아닌, 빈 컬렉션이나 배열을 반환하라

**컬렉션이 비었을 경우 null 처리**
```java 
List<Cat> cats = home.getCats();
if (cats != null && cats.getColor().equals("BLACK"))
    System.out.println("검정 고양이 있다!")
```
→ 위와 같은(cats != null) 방어코드를 꼭 넣어주어야하는 수고로움

`해결책`
1. null을 할당하는 것과 빈 컨테이너를 할당하는 것의 성능 차이는 거의 없다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

**→ 빈 컬렉션이나 배열을 반환하자!**

[컬렉션]
```java 
public List<Cat> getCats() {
   return new ArrayList<>(catList)
}
```

만약, 특정 상황에서 빈 컬렉션 할당이 성능을 크게 떨어뜨린다고 판단되는 경우에는 **매번 똑같은 빈 불변컬렉션을 반환**
```java 
public List<Cat> getCats() {
    return catList.isEmpty() ? Collections.emptyList() : new ArrayList<>(catList);
}
```

[배열]
```java 
public Cat[] getCats() {
    return catList.toArray(new Cat[0]);
}
```

만약 컬렉션 예시와 마찬가지로 새로운 배열을 계속 할당하는 것이 우려된다면, **배열을 미리 선언하고 그 배열을 반환하는 방식**
```java 
private static final Cat[] EMPTY_CAT_ARRAY = new Cat[0];

public Cat[] getCats() {
    return catList.toArray(EMPTY_CAT_ARRAY);
}
```
- toArray() : 주어진 배열의 크기와 같거나 더 큰 크기의 배열을 만들어서 리스트(catList)의 요소를 해당 배열에 복사, 주어진 배열 (EMPTY_CAT_ARRAY) 크기가 0이므로 새로운 크기가 0인 배열이 만들어집니다.

하지만, toArray()에 넘기는 **배열을 미리 할당하는 것은 권장하지 않음**
```java 
return catList.toArray(new Cat[catList.size()])
```

- 전자 : 주어진 배열의 원소가 하나라도 있다면 Cat[] 타입의 배열을 새로 생성해 반환하고, 원소가 0개면 EMPTY_CAT_ARRAY를 반환한다.
- 후자 : 크기가 동적으로 할당된 배열을 생성하여 전달 -> catList가 비어있다면 크기가 0인 배열이 반환된다.

오히려 성능이 떨어진다는 연구 결과가 있기 때문!

> https://shipilev.net/blog/2016/arrays-wisdom-ancients/
>  
> 
> 배열의 크기가 작을 경우 크기를 예측하는 과정에서 불필요한 오버헤드가 발생할 수 있기 때문에 오히려 성능에 부정적인 영향을 미칠 수 있다

## 결론
null 반환을 지양하고, 빈 컬렉션이나 빈 배열을 반환하자!

빈 컬렉션이나 빈 배열을 반환하는 여러가지 방법이 있으므로 성능과 메모리 이점을 비교하여 상황에 맞게 사용하자