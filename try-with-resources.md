일반적인 try-catch는 stream등이 활성화된 상태에서 exception이 발생할 경우 finally 문으로 close를 해줘야한다.

이는 가독성을 저하시키고 복잡한 코드로 작업을 번거롭게 만든다.

try-with-resources 는 예외가 발생할 경우 AutoCloseable 객체로 자원을 관리하여 에러처리를 용이하게 해준다.

JAVA는 기존에 사용하던 closeable 객체들을 AutoCloseable 로 바꿔주는 방법 대신 Closeable 인터페이스를 AutoCloseable객체를 상속 받도록 하여 간단하면서도 모든 Closeable객체가 AutoCloseable이 될 수 있도록 하였다.

```JAVA
try (AutoCloseableResources) {
	//실행코드...
}
```