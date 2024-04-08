## POJO
- Plain Old Java Object 의 약자
- 특정한 프레임워크나 라이브러리에 종속되지 않은 일반적인 자바 객체
## JAVA Bean
1. public no-arg constructor 가 있어야 함
2. getters 와 setters가 필수로 있어야 함
3. 인터페이스 Serializable 의 구현체여야만 함 
	- Serializable 의 구현체는 직렬화가 가능하다는 것을 나타냄
	- 직렬화란 객체를 바이트 스트림으로 변환하여 저장하거나 네트워크를 통해 전송할 수 있는 과정
	- 직렬화 하고 싶지 않은 필드는 private transient String sensitiveInformation; 과 같이 transient 를 붙여줘야 함


## Spring Bean

^803ea8

- Spring에 의해 관리되는 모든 객체
- Spring은 IOC container(Bean Factory or Application Context) 를 통해 객체를 관리