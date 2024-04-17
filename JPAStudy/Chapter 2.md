#### 객체 맵핑
##### @Entity
이 어노테이션이 붙은 클래스를 테이블과 매핑한다고 JPA에게 알려준다. 해당 어노테이션이 붙은 클래스를 Entity 클래스라고 한다.

##### @Table
엔티티 클래스를 매핑할 클래스의 정보를 알려준다.
name 속성을 이용하여 테이블 이름을 지정해줄 수 있다.

##### @Id
해당 필드를 테이블의 기본 키(primary key)에 매핑한다. @Id를 사용하는 필드를 식별자라고 한다.

##### @Column
필드를 컬럼에 매핑한다. name속성을 사용하여 매핑할 컬럼의 이름을 지정해줄 수 있다.

##### 매핑정보가 없는 필드
매핑어노테이션을 생략하면 필드명과 일치하는 컬럼에 매핑된다.


#### 데이터베이스 방언(DB Dialect)
JPA는 특정 DB에 종속적이지 않은 기술이다. 하지만 각 DB가 사용하는 문법과 함수가 조금씩 다르다. 
SQL표준을 지키지 않거나 특정 DB만의 고유한 기능을 JPA에서는 방언(Dialect)이라고 한다. DB를 바꾸게 되면 방언만 바꿔주면 되는것이다.

```xml
hibernate.show_sql : 하이버네이트가 실행한 SQL을 출력한다
hibernate.format_sql : 하이버네이트가 실행한 SQL을 출력할 때 보기 쉽게 정렬한다.
hibernate.use_sql_comments : SQL을 출력할 때 주석도 같이 출력한다.
hibernate.id.new_generator mappings : JPA 표준에 맞춘 새로운 키 생성 전략을 사용한다.
```

#### 엔티티 매니저 생성
##### 엔티티 매니저 팩토리 생성
JPA를 시작하기 위해 persistence.xml 의 설정정보를 사용해서 엔티티 매니저 팩토리를 생성해야 한다.
```JAVA
EntityManagerFactory emf =
	Persistence.createEntityManagerFactory("jpabook");
```
위 코드는 META-INF/persistence.xml 에서 이름이 jpabook인 영속성 유닛을 찾아서 엔티티 매니저 팩토리를 생성한다.
이 과정은 JPA동작시키기 위한 기반 객체를 만들고,  데이터베이스 커넥션 풀도 생성한다. 이는 비용이 매우 크다. 따라서 

**엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.**

##### 엔티티 매니저 생성
팩토리를 이용하여 엔티티 매니저를 생성한다.
```JAVA
EntityManager em = emf.createEntityManager();
```
엔티티 매니저는 JPA기능 대부분을 제공한다.
대표적으로 **등록/수정/삭제/조회** 를 할 수 있다. 엔티티 매니저는 내부에 데이터소스(데이터베이스 커넥션)를 유지하면서 데이터베이스와 통신한다. 즉 엔티티 매니저를 가상의 DB로 생각하며 개발 할 수 있다.
**엔티티 매니저는 데이터 베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안 된다.**

##### 종료
사용이 끝난 엔티티 매니저는 다음처럼 반드시 종료해야 한다.
```
em.close(); //엔티티 매니저 종료
```

애플리케이션 종료시에 엔티티 매니저도 종료해줘야 한다.
```JAVA
emf.close();
```




#### 트랜잭션 관리
트랜잭션 없이 데이터를 변경하면 예외가 발생한다. 트랜잭션을 시작하기 위해 엔티티 매니저에서 트랜잭션 API를 받아와야 한다.
```JAVA
EntityTransaction tx = em.getTransaction();
try {
	tx.begin();
	logic(em);
	tx.commit();
} catch (Exception e) {
	tx.rollback();
}
```
비즈니스 로직이 정상적으로 동작하면 트랜잭션을 커밋하고 예외가 발생하면 롤백 한다.

#### 비즈니스 로직
비즈니스 로직은 em을 통하여 수행된다. 엔티티 매니저는 객체를 저장하는 가상의 데이터베이스처럼 보인다.
```JAVA
public static void logic(EntityManager em) {  
  
        String id = "id1";  
        Member member = new Member();  
        member.setId(id);  
        member.setUsername("지한");  
        member.setAge(2);  
  
        //등록  
        em.persist(member);  
          
        //수정  
        member.setAge(20);  
  
        //한 건 조회  
        Member findMember = em.find(Member.class, id);  
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());  
  
        //목록 조회  
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();  
        System.out.println("members.size=" + members.size());  
  
        //삭제  
        em.remove(member);  
    }  
}
```
등록, 수정, 한 건 조회, 목록 조회, 삭제 를 하는 비즈니스 로직이다. 출력 결과는 다음과 같다.
```bash
findMember=지한, age=20
members.size=1
```
##### 등록
```JAVA
        String id = "id1";  
        Member member = new Member();  
        member.setId(id);  
        member.setUsername("지한");  
        member.setAge(2);  
  
        //등록  
        em.persist(member);  
```
엔티티를 저장하려면 엔티티 매니저의 persist() 메소드에 저장할 엔티티를 넘겨주면 된다.

##### 수정
```JAVA
        //수정  
        member.setAge(20);  
```
수정을 보면 em을 사용하지 않고 member의 값을 바꿔주고 있다. JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있다. 따라서 member객체의 값을 바꿔주면 UPDATE 쿼리가 실행된다.

##### 삭제
```JAVA
        //삭제  
        em.remove(member); 
```
엔티티를 삭제하기 위해선 em.remove() 메소드에 삭제하고자 하는 엔티티를 넣어준다.

##### 한 건 조회
```JAVA
        //한 건 조회  
        Member findMember = em.find(Member.class, id);  
```
find() 메소드는 조회할 엔티티 타입과 @Id로 데이터 베이스 테이블의 기본 키와 매핑한 식별자 값으로 하나의 엔티티를 조회하는 메소드이다.

#### JPQL(Java Persistence Query Language)
목록을 조회하는 코드를 살펴보자
```JAVA
        //목록 조회  
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();  
```
위에서 살펴본 등록, 수정,  삭제,  한 건 조회의 경우 SQL을 전혀 사용하지 않았다. 하지만 검색의 경우 em객체로만 처리를 하려면 DB의 모든 데이터를 em으로 가져와서 엔티티 객체로 변경 후  검색하게 되면 너무 비용이 크다. 따라서 필요한 데이터만 데이터베이스에서 가져오기 위해  검색 조건이 포함된 SQL을 사용한다. JPA는 JPQL을 사용하여 이 문제를 해결한다.

JPQL 은 SQL과 유사하며 SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등을 사용할 수 있다.
SQL과 JPQL의 차이는 다음과 같다
- JPQL은 엔티티 객체를 대상으로 쿼리한다. 클래스와 필드를 대상으로 쿼리한다.
- JPQL은 대소문자를 명확하게 구분한다.
- SQL은 데이터베이스 테이블을 대상으로 쿼리한다.
위 코드에서 
```JAVA
"select m from Member m"
```
이 부분이 JPQL이다. 중요한 점은 여기서 Member는 MEMBER테이블이 아닌 엔티티 객체를 말하는 것이다. JPQL은 DB의 테이블 정보를 전혀 모른다.JPQL을 사용하는 방법은 createQuery() 로 쿼리를 생성한 후 getResultList() 로 실행 결과를 가져온다.

### 정리
JPA가 반복적인 JDBC API와 결과 값 매핑을 처리해준 덕분에 코드량이 상당히 많이 줄어들었다. 그리고 SQL 도 작성할 필요가 없다.