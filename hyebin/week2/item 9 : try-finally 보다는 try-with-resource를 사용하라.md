## [아이템 9] try -finally 보다는 try -with -resources 를 사용하라

**꼭 회수해야 하는 자원을 다룰 때는 `try-with-resources`를 사용하자** 

자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어진다. 

`try- with- resources`의 핵심은 **AutoCloseable 인터페이스 구현이다.**  닫아야 하는 자원을 뜻하는 클래스를 작성한다면 AutoCloseable을 반드시 구현해야 한다.

```java
static void copy(String src, String dst) throws IOException{
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)){
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while((n == in.read(buf)) <= 0)
			out.write(buf, 0,n);
}
}
```

여기서 catch 문을 사용해서 try에서 발생할 수 있는 예외를 잡아준다면 close 호출에서 발생하는 예외와 try 에서 실행할 코드에서 발생하는 예외를 둘 다 잡을 수 있다는 장점이 있다.
