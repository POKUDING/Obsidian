#### DAO(DB Access Object)
이름의 뜻 그대로 데이터베이스에 접근하기 위한 객체이다. 
JDBC API를 사용하여 DB와의 통신을 담당한다.
DAO에 SQL 을 사용한 로직을 실행해주는 메소드를 정의하면 외부에서 DAO메소드를 이용하여 데이터베이스에 CRUD작업을 진행한다.

#### JPA의 필요성(JDBC사용 개발의 단점)
1. 반복적인 작업
	- 새로운 테이블이 필요하여 생성할 경우 CRUD와 필요한 Query를 모두 직접 작업해주어야 한다.
2. SQL 의존적 개발
	- 테이블에 새로운 열을 추가해야 할 경우 CRUD의 쿼리문을 모두 수정해주어야 한다.
	- 또한 조회 쿼리등이 수정될 경우 DAO의 메소드를 신뢰하여 사용하지 못 하고 DAO내부 SQL을 확인하며 작업을 진행하여야 한다.

#### JPA가 문제를 해결하는 방법
-  객체를 DB에 저장하고 관리할 때, SQL이 아닌 JPA의 API를 사용한다.
- 수정의 경우 JPA API 가져온 객체의 값을 수정하면 트랜잭션 커밋 시 수정이 적용된다.
- 연관 객체의 조회의 경우 연관 객체를 조회하는 시기에 적절한 SELECT SQL을 실행하여 가져온다.
```JAVA
//Create
jpa.persist(member); //저장

//Read
String memberId = "helloId"
jpa.find(Member.class, memberId); // 조회

//Update
Member member = jpa.find(Member.class, memberId);
member.setName("이름변경"); // 수정

//Read realation object
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam(); // 연관 객체 조회
```


#### 패러다임 불일치로 발생하는 문제
1. 데이터베이스는 정보중심으로 구조화 되어 있어서 객체지향에 존재하는 추상화, 상속, 다형성 을 지원하지 않는다.
	- JPA를 사용하게 되면 상속과 관련된 패러다임 불일치 문제를 해결해준다.
```JAVA
abstarct class Item {
	Long id;
	String name;
	int price;
}

class Album extends Item {
	String artist;
}

jpa.persist(album);
```
```SQL
	INSERT INTO ITEM ...
	INSERT INTO ALBUM ...
```
```JAVA
String albumId = "id100";
jpa.find(Album.class, albumId);
```
```SQL
SELECT I.*, A,*
	FROM ITEM I
	JOIN ALBUM A ON I.ITEM_ID = A.ITEM_ID
```
2. 객체는 참조를 사용하여 연관관계를 가진다. 테이블은 외래 키를 사용하여 연관관계를 가진다.
	- 연관관계를 가지고 있는 개체를 저장할 경우 참조를 PK 와 FK 관계로 DB에 저장해야한다.
	- 연관관계를 가지고 있는 데이터를 읽어올 경우  FK, PK를 참조로 변환해서 객체에 넣어주어야 한다.

이러한 과정은 패러다임 불일치 문제를 해결해주기 위한 소모비용이다. 만약 자바 컬렉션에 회원 객체를 저장한다면 이런 비용이 전혀 들지 않는다.

#### JPA를 사용하여 패러다임 불일치 문제를 해결
```JAVA
member.setTeam(team); // 회원과 팀 연관관계 설정
jpa.persist(member); // 회원과 연관관계 함께 저장
```
JPA는 연관관계를 PK와 FK로 설정하고 적절한 INSERT SQL을 DB에 전달한다.
```
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
```
죄회할 때 외래 키를 참조로 변환하는 일도 JPA가 처리해준다.

#### 객체의 그래프 탐색
```
Delivery - Order - Member - Team
			 |
		   OrderItem - Item - Category
```
다음과 같은 구조의 DB가 있다고 가정해보자.
DAO를 사용할 경우 Member 객체를 통해 어디까지 정보를 가져올 수 있을까?

답은 **처음 실행하는 SQL에 달려있다.**

처음 실행하는 SQL문이 어디까지 JOIN을 하여 가져올지 모르기에 DAO를 사용하는 입장에서는 DAO내부 구현을 확인하며 개발을 진행 할 수 밖에 없다.

그나마 해결 가능한 방안으로는
```JAVA
memberDAO.getMember();
memberDAO.getMemberWithTeam();
memberDAO.getMemberWithOrderWithDelivery
```
위와 같이 각 메소드를 다르게 정의해주는 것이다.

#### JPA에서의 그래프 탐색
JPA에서는 연관 데이터를 조회하는 상황에서 적절한 SELECT SQL문을 실행한다.
이를 **지연 로딩**이라 한다.
```JAVA
//처음 조회 시점에 SELECT MEMBER SQL
Member member = jpa.find(Member.class, memberId);

Order order = member.getOrder();
order.getOrderDate(); // Order를 사용하는 시점에 SELECT ORDER SQL
```
만약 Member를 조회시 Order를 같이 사용하는 경우가 많다고 하면  한테이블씩 조회하는것 보다 한번에 조회하는것이 효과적이다. jpa는 이러한 설정도 가능하다.


#### 비교
DAO에서는 같은 메소드로 조회한 객체가 서로 동일하지 않다.
```
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);

member1 == member2; // 다르다
```
JPA의 경우 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다.
```
String memberId = "100";

Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2; // 같다.
```


#### 결론
객체 모델과 데이터 모델이 지향하는 패러다임이 다르고 이를 해결하기 위해 소모해야하는 개발자의 리소스가 크다. 또한 정교한 객체 모델링은 패러다임간의 간극을 더 크게 만든다. 
**JPA를 사용하면 패러다임 불일치 문제를 해결해주고 정교한 객체 모델링을 유지하게 도와준다.**


#### JPA(Java Persistence API, 자바 영속성 API)란
 - 자바진영 ORM기술 표준
 - 자바 ORM 기술에 대한 API 표준 명세
 - 자바 애플리케이션과 JDBC사이에서 동작
 - ORM(Object-Relational Mapping) 객체와 관계형 DB 매퍼
 - JPA 구현 프레임워크
	 - 하이버네이트
	 - EclipseLink
	 - DataNucleus
