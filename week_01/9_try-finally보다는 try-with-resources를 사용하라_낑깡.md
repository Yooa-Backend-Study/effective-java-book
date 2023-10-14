#아이템9 | try-finally보다는try-with-resources를 사용하라
<hr>
자바 라이브러리에는 close 메서드를 호출하여 직접 닫아줘야 하는 자원이 많다.

자원 닫기가 필요한 라이브러리에는 예를 들어 아래와 같다.

+ 파일 I/O 관련 라이브러리
    + BufferedReader, BufferedWriter, FileReader, FileWriter: 파일을 읽거나 쓸 때 사용
    + FileInputStream, FileOutputStream: 이진 파일을 읽거나 쓸 때 사용
    + FileReader, FileWriter: 텍스트 파일을 읽거나 쓸 때 사용
+ 데이터베이스 연결 라이브러리
    + JDBC (Java Database Connectivity): 데이터베이스와 연결하고 쿼리를 실행할 때 사용
+ 네트워크 관련 라이브러리
    + Socket: 네트워크 소켓을 열고 데이터를 전송할 때 사용
+ 자원 관리 라이브러리
    + Apache Commons IO, Guava 등: 파일 및 스트림 관련 작업을 보다 쉽게 수행하고 자원을 자동으로 닫도록 도와줌

자주 쓰는 라이브러리들이지만 하위 코드가 길어지고, 분기 처리가 많아지다 보면 종종 자원 닫기를 놓치기 마련이다.

> 특히 데이터베이스를 연결할 때 Connection 객체를 생성하는데, 사용이 끝난 후 DB Connection을 제대로 close 하지 않으면 **Memory Leak, DB resource 부담, 트랜잭션 일관성 문제,  애플리케이션 안정성 문제** 등과 같은 문제를 발생시킬 수 있다.

이런 자원 중 상당수는 안전망으로 finalizer를 사용하고는 있지만, finalizer는 불확실성과 더불어 성능저하로 이어진다.

### try-finally
이에 전통적으로 자원이 제대로 닫힘을 보장해 줄 수 있는 수단으로 `try-finally`가 쓰였다.

- 기본 try-finally : 더 이상 자원을 회수하는 최선의 방책이 아니다!
``` java
static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```

- 자원이 둘 이상이라면 ? (직접 예제 구현)
``` java
** tryfinally.java
public class tryfinally {
    // 코드 9-2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! (47쪽)
    // 코드 9-2 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다.
    static void test(String arg) throws IOException {
        TestFileInputStream in = new TestFileInputStream();
        try {
            TestFileOutputStream out = new TestFileOutputStream();
            try {
                String inData = in.write(arg); // 여기서 먼저 에러가 났음에도 불구하고
                out.print(inData);
            } finally {
                out.close();
            }
        } finally {
            in.close(); // 여기서의 에러만 남긴다.
        }
    }

    public static void main(String[] args) throws IOException {
        test("");
    }
}

---

** TestFileInputStream.java
public class TestFileInputStream {
    public String write(String writeData) throws IOException {
        System.out.println("[TestFileInputStream] write : " + writeData);

        if (writeData.isEmpty())
            throw new IOException("[TestFileInputStream] No Data");
        else
            return writeData;
    }

    public void close() throws IOException {
        throw new IOException("[TestFileInputStream] close exception");
    }
}

---

** TestFileOutputStream.java
public class TestFileOutputStream {
    public String print(String inData) throws IOException {
        System.out.println("[TestFileOutputStream] in data : " + inData);
        if (inData.isEmpty())
            throw new IOException("[TestFileOutputStream] No Data");
        else return inData;
    }

    public void close() throws IOException {
        throw new IOException("[TestFileOutputStream] close exception");
    }
}
```

> [TestFileInputStream] write :
**Exception in thread "main" java.io.IOException: [TestFileInputStream] close exception**
at item09.tryfinally.TestFileInputStream.close(TestFileInputStream.java:17)
at item09.tryfinally.tryfinally.test(tryfinally.java:19)
at item09.tryfinally.tryfinally.main(tryfinally.java:24)

앞의 예외들이 명확히 발생했음에도 불구하고, 마지막 예외인 TestFileInputStream의 close Exception만 출력되고 있다.

일반적으로 발생한 문제를 진단하려면 처음 발생한 예외를 보아야 하는데, 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않아 디버깅을 몹시 어렵게 한다.

> 위와 같은 문제가 발생하는 이유는 **예외 처리의 동작 방식**과 **try-finally의 우선순위** 때문이다.
>
>try-finally 블록은 오류가 발생하든 발생하지 않든 항상 실행되므로, 첫 번째 예외가 발생하고 finally 블록에서 예외가 또 발생하면 두 번째 예외가 우선시 되어 첫 번째 예외는 무시되어 프로그램은 두 번째 예외를 처리하게 된다.

(처음 발생한 예외를 기록하도록 코드를 수정할 수도 있겠지만 번거롭고 코드가 지저분해진다.)

### try-with-resources
이러한 문제들을 자바 7의 try-with-resources를 사용하여 해결할 수 있다.

try-with-resources를 사용하려면
1. 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.

- 복수의 자원을 사용하는 try-finally 코드를 개선해 보았다. (직접 예제 구현)
```java 
** trywithresources.java
public class trywithresources {
    static String test(String arg) throws IOException {
        try ( TestFileInputStream in = new TestFileInputStream();
              TestFileOutputStream out = new TestFileOutputStream()) {
            String inData = in.write(arg); // 가장 먼저 발생한 예외
            return out.print(inData);
        }
    }

    public static void main(String[] args) throws  IOException {
        String result = test("");
        System.out.println("[MAIN] result : " + result);
    }
}

---
** TestFileInputStream.java
public class TestFileInputStream implements AutoCloseable{
    public String write(String writeData) throws IOException {
        System.out.println("[TestFileInputStream] write : " + writeData);

        if (writeData.isEmpty())
            throw new IOException("[TestFileInputStream] No Data"); // 가장 먼저 발생한 예외
        else
            return writeData;
    }

    @Override
    public void close() throws IOException {
        throw new IOException("[TestFileInputStream] close exception");
    }
}

---
** TestFileOutputStream.java
public class TestFileOutputStream implements AutoCloseable {
    public String print(String inData) throws IOException {
        System.out.println("[TestFileOutputStream] in data : " + inData);
        if (inData.isEmpty())
            throw new IOException("[TestFileOutputStream] No Data");
        else return inData;
    }

    @Override
    public void close() throws IOException {
        throw new IOException("[TestFileOutputStream] close exception");
    }
}
```

> [TestFileInputStream] write :
Exception in thread "main" java.io.IOException: [TestFileInputStream] No Data
**at item09.trywithresources.TestFileInputStream.write(TestFileInputStream.java:10)**
at item09.trywithresources.trywithresources.test(trywithresources.java:10)
at item09.trywithresources.trywithresources.main(trywithresources.java:20)
>
> **Suppressed: java.io.IOException: [TestFileOutputStream] close exception**
at item09.trywithresources.TestFileOutputStream.close(TestFileOutputStream.java:15)
at item09.trywithresources.trywithresources.test(trywithresources.java:8)
... 1 more
>
> **Suppressed: java.io.IOException: [TestFileInputStream] close exception**
at item09.trywithresources.TestFileInputStream.close(TestFileInputStream.java:17)
at item09.trywithresources.trywithresources.test(trywithresources.java:8)
... 1 more

가장 먼저 발생한 예외가 스택 추적 내역에 기록되고, 그다음 finally에서 발생한 예외들은 무시되지 않고 '숨겨졌다(Suppressed)' 태그를 달고 출력된다.

또한, 자바 7에서 Throwable에 추가된 **getSuppressed 메서드**를 이용하면 프로그램 코드에서 가져올 수도 있으며 **catch 절**을 사용해 try 문을 중첩하지 않고도 다수의 예외를 처리할 수 있다!

- (직접 구현 예제)
```java
public class trywithresources {
    static String test(String arg){
        try ( TestFileInputStream in = new TestFileInputStream();
              TestFileOutputStream out = new TestFileOutputStream()) {
            String inData = in.write(arg);
            return out.print(inData);
        }
        catch (IOException e){ // 추가된 코드들
            System.out.println(Arrays.stream(e.getSuppressed()).findFirst()); // 가장 첫 번째로 숨겨진 예외를 출력
            return "defaultValue"; //
        }
    }

    public static void main(String[] args) {
        String result = test("");
        System.out.println("[MAIN] result : " + result);
    }
}
```
> [TestFileInputStream] write :
>
> Optional[java.io.IOException: [TestFileOutputStream] close exception]
>
> [MAIN] result : defaultValue

먼저 정의되거나 획득된 리소스는 마지막에 닫힌다. (LIFO 원칙)



**추가로 자바 9부터는 final 변수를 효과적으로 사용할 수 있다.**
자바 9 전에는, try-with-resources  블록 내에서만 새로운 변수를 사용할 수 있었다.
```java 
try (Scanner scanner = new Scanner(new File("testRead.txt"));
    PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) {
    // omitted
}
```
위와 같이 여러 리소스를 사용한다면, 코드가 장황해지고 가독성이 떨어진다.

Java 9 및 JEP 213 의 일부로 이제 try-with-resources 블록 내에서 final 또는 효과적으로 final 변수를 사용할 수 있다.
```java 
final Scanner scanner = new Scanner(new File("testRead.txt"));
PrintWriter writer = new PrintWriter(new File("testWrite.txt"))
try (scanner;writer) {
    // omitted
}
```
- 명시적으로 final로 표시되지 않은 경우에도 처음 할당 후 변경되지 않으면 해당 변수는 사실상 final이다. 따라서 writer 변수도 처음 할당 후 변경되지 않으므로 writer 변수 또한 try-with-resources 블록에서 사용할 수 있다.


