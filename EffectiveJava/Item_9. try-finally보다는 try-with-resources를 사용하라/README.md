# Item 9. try-finally보다는 try-with-resources를 사용하라

 자바 라이브러리에는 close() 메서드를 호출하여 직접 닫아줘야 하는 자원이 많습니다. InputStream, OutputStream, java.sql.Connection 등이 좋은 예입니다. 이러한 자원들을 닫을 때는 흔하게 사용하는 try-finally 보다는 try-with-resource를 사용하는 것이 더 간결하고 예외 상황에서 문제점을 파악하기 좋습니다.

 try-finally 와 try-with-resources 예시를 함께 보며 비교해 보겠습니다.

#### try-finally

**firstLineOfFile() 메서드**

```
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(PATH));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```

**copy() 메서드**

```
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```

 두 메서드는 finally 에서 자원을 닫기 위해 close() 메서드를 호출하고 있습니다. 하지만 예외는 try 블록뿐만 아니라 finally 블록에서도 발생할 수 있습니다. 예를 들어, 기기에 물리적인 문제가 생긴다면 firstLineOfFile() 메서드 안의 readLine() 메서드가 예외를 던지고, 같은 이유로 close() 메서드도 실패할 것입니다. 이런 상황이면 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버립니다. 따라서 스택 트레이스에 첫 번째 예외에 관한 정보가 남지 않게 되어, 디버깅을 어렵게 합니다.

#### try-with-resources

**firstLineOfFile() 메서드**

```
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(PATH))) {
            return br.readLine();
        }
    }
```

**copy() 메서드**

```
    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
```

 처음에 살펴보았던 try-finally 블록에 비해 코드가 간결합니다. 이러한 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 합니다. try-with-resources 구문은 readLine()와 close()의 양쪽 호출에서 예외가 발생하면, close()에서 발생한 예외는 숨겨지고 readLine() 에서 발생한 예외가 기록됩니다.

 try-with-resources는 try-finally에서처럼 catch 절을 사용할 수 있습니다.

---

## 참고자료

[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LAG&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LAG&Kc=)
