## 영속성 관리
### 엔티티를 엔티티 매니저를 통해 사용하기
#### 엔티티 매니저와 엔티티 매니저 팩토리
데이터베이스를 하나만 사용하는 애플리케이션은 하나의 엔티티 매니저 팩토리만 사용한다.
```JAVA
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook")
```
이 코드는 META-INF/persistence.xml에 있는 정보를 바탕으로 EntitiManagerFactory를 생성한다.
```xml
<persistence-unit name="jpabook">  
    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>  
    <class>jpabook.start.Member</class>  
    <properties>  
        <!-- 필수 속성 -->  
        <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>  
        <property name="javax.persistence.jdbc.user" value="sa"/>  
        <property name="javax.persistence.jdbc.password" value=""/>  
        <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:db"/>  
        <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />  
  
        <!-- 옵션 -->  
        <property name="hibernate.show_sql" value="true" />  
        <property name="hibernate.format_sql" value="true" />  
        <property name="hibernate.use_sql_comments" value="true" />  
        <property name="hibernate.id.new_generator_mappings" value="true" />  
  
        <property name="hibernate.hbm2ddl.auto" value="create" />  
    </properties></persistence-unit>
```

이제부터 필요할 때 마다 엔티티 매니저 팩토리를 통해 엔티티 매니저를 생성하면 된다.
```JAVA
EntityManager em = emf.createEntityManager();
```
엔티티 매니저 팩토리는 만드는데 비용이 크다. 따라서 한개의 팩토리를 만들어서 애플리케이션 전체에서 공유해서 사용하도록 설계되어 있다.

반면 엔티티 매니저는 생성비용이 거의 없고 **엔티티 매니저 팩토리는 스레드간 공유에 안전하지만 엔티티 매니저는 스레드간 공유시 동시성 문제가 발생하므로 스레드 간에 절대 공유하면 안된다.**

엔티티 매니저는 꼭 필요한 시점까지 데이터베이스 커넥션을 사용하지 않는다. 데이터베이스 커넥션을 획득하는 시기는 예로 트랜잭션을 시작할 때 획득한다.

하이버네이트를 포함한 JPA구현체들은 엔티티 매니저 팩토리를 생성할 때 커넥션풀도 만든다. 이는 J2SE환경에서 사용하는 방법이다. 
J2EE 환경에서는 해당 컨테이너가 제공하는 데이터소스를 사용한다.


### 영속성 컨텍스트란?
영속성 컨텍스트(persistence context) 는 "엔티티를 영구 저장하는 환경" 이라는 뜻으로 JPA를 이해하는데 가장 중요한 용어이다.
엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다. 즉
```JAVA
em.persist(member);
```
이 코드는 **엔티티 매니저를 사용해서 회원 엔티티를 영속성 컨텍스트에 저장**하는 코드이다.

영속성 컨텍스트는 논리적인 개념에 가깝고 눈에 보이지도 않는다. 영속성 컨텍스트는 엔티티 매니저를 생성할 때 하나 만들어지며 엔티티 매니저를 통해서 영속성 컨텍스트에 접근할 수 있고, 영속성 컨텍스트를 관리할 수 있다.
> 여러 엔티티 매니저가 같은 영속성 컨텍스트에 접근할 수도 있다.

### 엔티티의 생명주기
엔티티에는 4가지 상태가 존재한다.
- 비영속(new/transient): 영속성 컨텍스트와 전혀 관계가 없는 상태
- 영속(managed): 영속성 컨텍스트에 저장된 상태
- 준영속(detached): 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed): 삭제된 상태

#### 비영속
엔티티 객체를 생성하면 순수한 객체 상태이며 아직 저장하지 않았다. 따라서 영속성 컨텍스트나 데이터베이스와는 전혀 관련이 없다. 이를 비영속상태라 한다.
```JAVA
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```

#### 영속
엔티티 매니저를 통해서 엔티티를 영속성 컨텍스트에 저장하면 **영속성 컨텍스트가 엔티티를 관리하게 되며 이를 영속상태라고 한다. 즉 영속 상태는 영속성 컨텍스트에 의해 관리된다는 뜻이다.**
```JAVA
em.persist(member);
```

#### 준영속
영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면 준영속 상태가 된다.
**em.detach() 를 호출하거나 em.close()를 호출해서 영속성 컨텍스트를 닫거나  em.clear()를 호출해서 영속성 컨텍스트를 초기화**해도 영속성 컨텍스트가 관리하던 **영속 상태의 엔티티는 준영속 상태**가 된다.
```JAVA
em.detach(member);
```

#### 삭제
엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다.
```JAVA
em.remove(member);
```


### 영속성 컨텍스트의 특징
- 영속성 컨텍스트와 식별자 값
	 영속성 컨텍스트는 엔티티를 식별자 값(@Id)로 구분한다. 따라서 **영속상태는 식별자 값이 반드시 있어야 한다.** 식별자 값이 없으면 예외가 발생한다.
	 
- 영속성 컨텍스트와 데이터베이스 저장
	영속성 컨텍스트에 엔티티를 저장하면 언제 DB에 저장될까? JPA는 보통 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 데이터베이스에 반영한다. 이것을 **플러시(flush)**라 한다.
	
- 영속성 컨텍스트가 엔티티를 관리하는 경우의 장점
	- 1차 캐시
	- 동일성 보장
	- 트랜잭션을 지원하는 쓰기 지연
	- 변경 감지
	- 지연 로딩

#### 영속성 컨테이너 사용의 이점
##### 조회
영속성 컨텍스트는 내부에 캐시를 가지고 있는데 이것을 1차 캐시라 한다. 영속 상태의 엔티티는 모두 이곳에 저장된다.
```JAVA
//엔티티를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

//엔티티를 영속
em.persist(member);
```
이 코드를 실행하면 영속성 컨테이너에 엔티티가 영속되며 1차 캐시에 엔티티를 저장한다. 
**회원 엔티티는 아직 데이터베이스에 저장되지 않았다.**

1차 캐시의 키는 식별자 값이다. 그리고 식별자 값은 데이터베이스 기본 키와 매핑되어 있다. 따라서 영속성 컨텍스트에 ㅈ데이터를 저장하고 조회하는 모든 기준은 데이터베이스 기본 키 값이다.

엔티티를 조회해보자.
```JAVA
Member member = em.find(Member.class, "member1");
```
**find()메소드의 첫 파라미터는 엔티티 클래스의 타입이고 두 번째는 조회할 엔티티의 식별자 이다.**
em.find()를 호출하면 먼저 1차 캐시를 조회하고 없으면 데이터베이스에서 조회한다.

**즉 데이터를 조회할 때 1차 캐시를 먼저 확인하기 때문에 데이터베이스와의 통신을 최소화 하여 성능상 이점을 가져올 수 있다.**

###### 영속 엔티티의 동일성 보장
```JAVA
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println(a == b); // a와 b의 동일성 검사
```
이 코드의 결과는 true이다.같은 식별자 값으로 여러번의 em.find() 호출을 진행하여도 1차 캐시에 있는 엔티티를 반환하기 때문에 결과는 항상 참이다. 

**즉 영속성 컨텍스트는 성능상 이점과 엔티티의 동일성을 보장한다.**
>동일성: 실제 인스턴스가 같다(주소값이 같다.)
>동등성: 인스턴스는 다르지만 같은 값으로 취급된다. (equals()) 메소드로 비교한다.

>JPA는 1차 캐시를 통해 반복가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다는 장점이 있다.

#기술부채 트랜잭션 격리 수준, 1차캐시에서 조회 할때 실 DB랑 다른 정보를 조회하면 어떡하지?



### JPA의 CRUD
##### 엔티티 등록
```JAVA
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
//엔티티 매니저는 데이터 변경 시 트랜잭션을 시작해야 한다.
transaction.begin();// [트랜잭션] 시작

em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit();// [트랜잭션] 커밋
```
엔티티 매니저를 통해 엔티티를 영속성 컨텍스트에 등록했다.
엔티티 매니저는 트랜잭션을 머밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 내부 쿼리 저장소에 INSERT SQL을 모아둔다. 그리고 트랜잭션을 커밋할 때 모아둔 쿼리를 데이터베이스에 보내는데 이것을 **트랜잭션을 지원하는 쓰기 지연(trasactional write-behind)** 이라한다.

트랜잭션을 커밋하면 엔티티 매니저는 우선 영속성 컨텍스트를 플러시한다.
플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화 하는 작업이다. 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화한 후에 실제 데이터베이스 트랜잭션을 커밋한다.

##### 엔티티 수정
###### 변경 감지(dirty checking)
```JAVA
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
transaction.begin(); //[트랜잭션] 시작

//영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

//영속 엔티티 데이터 수정
memberA.setUsername("hi");
memberA.setAge(10);

//em.update(memberA); 없어도 된다!!

trasaction.commint();
```
엔티티를 수정할 때는 단순히 값만 변경하면 된다. em.update() 와 같은 코드는 필요 없다. 어째서 일까?

JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장해두는데 이것을 스냅샷이라 한다. 그리고 플러시 시점에 스냅샷과 엔티티를 비교해서 변경된 엔티티를 찾는다.
1. 트랜잭션을 컷밋하면 엔티티 매니저 내부에서 먼저 플러시가 호출된다.
2. 엔티티와 스냅샷을 비교해서 변경된 엔티티를 찾는다.
3. 변경된 엔티티가 있으면 수정 쿼리를 생성해서 쓰기 지연 SQL 저장소에 보낸다.
4. 쓰기 지연 저장소의  SQL을 데이터베이스에 보낸다.
5. 데이터베이스 트랜잭션을 커밋한다.
이러한 과정을 통해 엔티티의 변경을 감지하여 데이터베이스에 자동으로 반영하는 것을 **변경 감지** 라고 한다.
물론, 변경감지는 영속성이 유지되고 있는 엔티티에서만 적용된다.

###### JPA의 수정 전략
JPA는 수정된 필드가 발견되면 해당 데이터만 반영할 것으로 예상되지만 모든 필드를 수정에 반영한다.
```SQL
%% 기대값 %%
UPDATE MEMBER
SET
	NAME =?,
	AGE=?
WHERE
	id=?

%% 실제 동작 %%
UPDATE MEMBER
SET
	NAME=?,
	AGE=?,
	GRADE=?,
	...
WHERE
	id=?
```
데이터 전송량이 증가하는 단점이 있음에도 이러한 이유는 무엇일까?
- 모든 필드를 사용하면 수정쿼리가 항상같다. 따라서 애플리케이션 로딩 시점에 수정 쿼리를 미리 생성해두고 재사용할 수 있다.
- 데이터베이스에 동일한 쿼리를 보내면 데이터베이스는 이전엔 한 번 파싱된 쿼리를 재사용할 수 있다.

다만, 필드가 많거나 저장되는 내용이 너무 크면 수정된 데이터만 사용해서 동적으로 UPDATE SQL을 생성하는 전략을 선택하면 된다.
```JAVA
@Entity
@org.hibernate.annotations.DynamicUpdate
@Table(name = "Member")
public class Member {...}
```

###### @org.hibernate.annotations.DynamicUpdate
이 어노테이션을 사용하면 수정된 데이터만 사용해서 동적으로 UPDATE SQL을 생성한다.
**@DynamicInsert**
이 어노테이션은 null이 아닌 필드만 INSERT SQL을 동적으로 생성한다.

>컬럼이 대략 30개 이상이 되면 기본 방법인 정적 수정 쿼리보다 @DynamicUpdate를 사용한 동적 수정 쿼리가 빠르다고 한다. 물론 직접 테스트 해보고 적용하는것이 좋다.

##### 엔티티 삭제
```JAVA
Member memberA = em.find(Member.class, "memberA");
em.remove(memberA);
```
이 코드는 memberA를 찾아서 삭제 하는 코드이다. 삭제도 수정과 마찮가지로 즉시 반영되는 것이 아닌 삭제 쿼리를 SQL 쓰기 지연 저장소에 등록한다. 이후 트랜잭션 커밋해서 플러시를 호출하면 실제 데이터베이스에 삭제 쿼리를 전달한다.

참고로 em.remove() 로 삭제한 엔티티는 영속성 컨텍스트에서도 제거된다. 이렇게 삭제된 엔티티는 재사용하지말고 자연스럽게 가비지 컬렉션의 대상이 되도록 두는 것이 좋다.

### 플러시
**플러시는 영속성 컨텍스트의 변경내용을 데이터 베이스에 반영한다.**
##### 플러시를 실행하면 일어나는 일
1. 변경 감지가 동작하여 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾는다. 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
2. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.(등록,수정,삭제 쿼리).

#### 영속성 컨텍스트를 플러시하는 방법
##### 직접호출
em.flush() 를 직접 호출한다. 테스트나 다른 프레임워크와 JPA를 함께 사용할 때를 제외하고 거의 사용하지 않는다.

##### 트랜잭션 커밋 시 플러시 자동 호출
데이터베이스에 변경 내용을 SQL로 전달하지 않고 트랜잭션만 커밋하면 어떤 데이터도 데이터베이스에 반영되지 않는다. 따라서 트랜잭션을 커밋하기 전에 꼭 플러시를 호출해서 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영해야 한다. JPA는 이런 문제를 예방하기 위해 트랜잭션을 커밋할 때 플러시를 자동으로 호출한다.

##### JPQL 쿼리 실행 시 플러시 자동 호출
JPQL이나 Criteria 같은 객체지향 쿼리를 호출할 때도 플러시가 실행된다. 그 이유는 JPQL을 사용하여 DB에 내용을 조회 할 때 영속성 컨텍스트에 있는 내용은 DB에 아직 반여되어 있지 않기 때문에 잘 못 된 데이터를 조회하는 문제가 생긴다. 이를 방지하기 위해 JPQL쿼리 실행시 플러시를 자동호출한다.
#기술부채 Criteria

#### 플러시 모드 옵션
```JAVA
em.setFlushMode(FlushModeType.AUTO);
em.setFlushMode(FlushModeType.COMMIT);
```
위 코드로 플러시 모드를 변경할 수 있다. 기본 설정은 AUTO로 커밋이나 쿼리실행시 플러시가 실행되고 COMMIT은 커밋할 때만 플러시가 일어난다.
### 준영속
**영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 것을 준영속 상태라 한다.**
엔티티를 준영속 상태로 만드는 방법은 크게 3가지다.
- detach()메소드 사용
	- 특정 엔티티를 준영속 상태로 바꾼다. 
- clear() 메소드 사용
	- 영속성 컨텍스트를 초기화 하여 기존 영속성 컨텍스트에서 관리하던 모든 엔티티를 준영속 상태로 만든다.
- close() 메소드 사용
	- 영속성 컨텍스를 종료하여 영속상태의 엔티티가 모두 준영속 상태가 된다.

##### 준영속 상태가 되면 어떤 변화가 생길까?
1. 영속성 컨텍스트에서 해당 엔티티에 대한 모든 정보를 삭제한다.
	1. 모든 정보는 1차 캐시와 쓰기 지연 SQL 저장소에 있는 모든 쿼리를 포함한다.
2. 준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.
3. 지연로딩을 할 수 없다.
4. 사실상 비영속 상태와 같다.(준영속 상태는 영속 상태였었기 때문에 식별자 값을 갖고 있다.)

#### 병합
**merge()** 메소드는 준영속 상태의 엔티티를 받아서 영속성 컨텍스트에 등록하고 **새로운 영속상태의 엔티티를 반환한다.**

##### 준영속 상태에서 엔티티의 값을 수정하고 병합했을 경우 어떤 작업이 일어날까?
1. 영속성 컨텍스트에 엔티티의 정보가 없기 때문에 DB에서 정보를 가져와서 1차 캐시에 등록한다.
2. 1차 캐시에 등록한 새로운 엔티티를 반환한다.
3. 새로운 엔티티에 merge 요청이 들어온 entity의 값으로 덮어 씌운다.
4. 새로운 엔티티를 반환한다.

따라서 준영속 상태에서 값을 수정하고 merge할 경우 새로운 영속상태의 엔티티에는 수정이 반영되어서 나온다.

### 정리
1. J2SE 환경에서는 엔티티 매니저를 만들면 그 내부에 영속성 컨텍스트도 함께 만들어진다. J2EE환경은 해당 컨테이너가 제공한는 데이터 소스를 사용한다.
2. 영속성 컨텍스트는 애플리케이션과 DB사이에서 객체를 보관하고 관리하는 가상의 DB같은 역활을 한다.
3. 영속성 컨텍스트 덕분에 1차캐시, 동일성 보장, 트랙잰션을 지원하는 쓰기 지연, 변경 감지, 지연 로딩 기능을 사용할 수 있다.
4. 영속성 컨텍스트에 저장한 엔티티는 플러시 시점에 데이터베이스에 반영되는데 일반적으로 트랜잭션을 커밋할 때 영속성 컨텍스트가 플러시된다.
5. 영속성 컨텍스트가 관리하는 엔티티를 영속 상태의 엔티티라고 하는데 더이상 영속성 컨텍스트가 관리하지 못 하면 이는 준영속 상태라고 한다.
6. 준영속 상태의 엔티티는 영속성 컨테이너의 관리를 받지 못 하기 때문에 영속성 컨테이너의 모든 기능을 사용할 수 없다.