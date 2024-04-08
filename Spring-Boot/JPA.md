### H2 database
pom.xml을 통해 starter 에 h2를 추가해주면 h2db를 사용할 수 있게 된다.
```xml
<dependency>  
    <groupId>com.h2database</groupId>  
    <artifactId>h2</artifactId>  
    <scope>runtime</scope>  
</dependency>
```

/h2-console 주소에 접속하게 되면 db url을 이용하여 db에 접속이 가능한데 이를 위해서는 h2-console에 접속할 수 있는 설정을 해줘야 한다.
또한 실행시에 db의 url을 찾을 수 있는데 이는 매번 실행할 때 마다 바뀌게 된다. 내가 원하는 url로 고정 시키기 위해서는 application.properties 파일에 설정해줘야 한다.
```properties
spring.h2.console.enabled=true  
spring.datasource.url=jdbc:h2:mem:testdb
```

실행시 기본적으로 실행시키고자 하는 sql문이 있을경우 resource폴더내에 schema.sql로 작성해 두면된다.
```sql
create table course  
(  
    id bigint not null,  
    name varchar(255) not null ,  
    author varchar(255) not null,  
    primary key (id)  
);
```
다음과 같이 설정한 후 h2-console에 접속해서 select * from couse sql 문을 실행시키면 테이블이 생성된 것을 확인할 수 있다.


### JDBC, Spring JDBC
데이터의 CRUD를 위해서는 SQL을 작성해야한다. 다음과 같은 SQL이 필요하다고 예시를 들자
```SQL
insert into course(id, name, author)
values(1, 'Learn AWS', 'in28minutes');

select * from course;

delete from course where id=1;
```
이를 Java 코드로 객체로써 접근하게 해주는 것이 JDBC와 Spring JDBC이다.


JDBC 로 insert 문을 실행시키는 방버이다.
```JAVA
@Repository  
public class CourseJdbcRepository {  
  
    @Autowired  
    JdbcTemplate springJdbcTemplate;  
  
    private static String INSERT_QUERY =  
            """  
                insert into course(id, name, author)  
                values(1, 'Learn AWS', 'in28minutes');  
            """;  
  
    public void insert() {  
        springJdbcTemplate.update(INSERT_QUERY);  
    }  
}
```
query 문을 INSERT_QUERY 변수에 넣어서 SpringJdbcTemplate 를 이용하여 실행시켰다.
insert함수가 실행이 되어야 하는데 시작하자마자 바로 실행할 수 있게 해주는 방법이 무엇일까?

```JAVA
@Component  
public class CouresJdbcCommandLineRunner implements CommandLineRunner {  
  
    @Autowired  
    private CourseJdbcRepository repository;  
    @Override  
    public void run(String... args) throws Exception {  
        repository.insert();  
    }  
}
```
CommandLineRunner를 구현한 CourseJdbcCommandLineRunner 를 @Component 어노테이션을 통하여 Spring Bean으로 만들어 주었다.

JAVA Spring은 맨처음
```JAVA
@SpringBootApplication  
public class LearnJpaHibernateApplication {  
  
    public static void main(String[] args) {  
       SpringApplication.run(LearnJpaHibernateApplication.class, args);  
    }  
  
}
```
run을 실행할 때 CommandLineRunner Bean과 ApplicationRunner Bean을 모두 불러와서 run()메소드를 실행시켜준다.

하지만 지금 코드는 다른 입력값을 받을 수 없다. 받기 위해서는 새로운 query 를 String으로 만들고 넣어주어야 하는데 너무 복잡하고 아름답지 못 하다. 확장에 닫혀있다....

INSERT_QUERY문을 바꿔주었다
```JAVA
private static String INSERT_QUERY =  
        """  
            insert into course(id, name, author)  
            values(?, ?, ?);  
        """;
```
넣어주어야 하는 입력값이? 로 비워져 있다.

Course class를 새로만들어 주었다.
```JAVA
public class Course {  
    private long id;  
    private String name;  
    private String author;  
  
    public Course() {}  
  
    public Course(long id, String name, String author) {  
        this.id = id;  
        this.name = name;  
        this.author = author;  
    }  
  
    public long getId() {  
        return id;  
    }
    ...
```
Course class 는 TABLE이 원하는 모든 값들을 가지고 있다.

insert함수는 다음과 같이 수정해주었다.
```JAVA
public void insert(Course course) {  
    springJdbcTemplate.update(INSERT_QUERY, course.getId(), course.getName(), course.getAuthor());  
}
```
course 객체를 매개변수로 받아 query문과 함께 매개변수로 넘겨주는 것을 볼 수 있다. 이는 순서대로 ? 자리에 대입되게 된다.(?의 갯수와 매개변수의 수가 다를 경우 서버가 실행되지 않는다.)

이후 CommandLineRunner 구현체의 run() 메소드를
```JAVA
repository.insert(new Course(1, "Learn AWS", "in28minutes"));
```
다음과 같이 수정해주게 되면 원하는 값을 유연하게 넣을 수 있는 설계가 된다.


조회가 조금 더 복잡했다.
id를 기준으로 sql을 조회해 보자. 우선 SQL문을 만들어 주었다.
```JAVA
private static String SELECT_QUERY =  
        """  
                select * from course where id = ?  
                """;
```
이제 ? 에 우리가 원하는 id를 넣으면 된다는 것을 알 수 있다.
메소드를 만들어 보자
```JAVA
public Course findById(long id) {  
    return springJdbcTemplate.queryForObject(SELECT_QUERY, new BeanPropertyRowMapper<>(Course.class),  id);  
}
```
new BeanPropoertyRowMapper<>(Course.class) 이 부분이 너무 헷갈렸다. 조금 더 깊게 공부를 해봐야 겠지만 지금까지 이해한 바로는 조회한 쿼리문의 결과를 어떤 형식으로 리턴할지를 정해주는 인자라고 보면될것 같다. Course.class에 우리는 열이름과 동일한 id, name, author가 있기 때문에 적절하게 맵핑하여 리턴해주는 것이다.
세번째 인자 부터 ?에 맵핑될 것이다.


### JPA!
기존 Jdbc 코드를 보자
```JAVA  
@Repository  
public class CourseJdbcRepository {  
  
    @Autowired  
    JdbcTemplate springJdbcTemplate;  
  
    private static String INSERT_QUERY =  
            """  
                insert into course(id, name, author)  
                values(?, ?, ?);  
            """;  
  
    private static String DELETE_QUERY =  
            """  
                    delete from course where id=?  
                    """;  
  
    private static String SELECT_QUERY =  
            """  
                    select * from course where id = ?  
                    """;  
  
    public void insert(Course course) {  
        springJdbcTemplate.update(INSERT_QUERY, course.getId(), course.getName(), course.getAuthor());  
    }  
  
    public void deleteById(long id) {  
        springJdbcTemplate.update(DELETE_QUERY,id);  
    }  
  
    public Course findById(long id) {  
        return springJdbcTemplate.queryForObject(SELECT_QUERY, new BeanPropertyRowMapper<>(Course.class),  id);  
    }  
}
```

로직의 처리는 깔끔하지만 query문을 관리하는 것이 복잡하다. 다뤄야할 쿼리문이 많아지면 더욱 복잡해질 것이다. JPA에 로직을 옮기면 훨씬 깔끔하다.

```JAVA
@Repository  
@Transactional  
public class CourseJpaRepository {  
  
    @PersistenceContext  
    private EntityManager entityManager;  
  
    public void insert(Course course) {  
        entityManager.merge(course);  
    }  
  
    public Course findById(long id) {  
        return entityManager.find(Course.class, id);  
    }  
  
    public void deleteById(long id) {  
        Course course = entityManager.find(Course.class, id);  
        entityManager.remove(course);  
    }  
}
```
훨씬 깔끔한 것을 알 수 있다.

@PersistenceContext 라는 새로운 어노테이션을 볼 수 있다. 이 어노테이션은 @Autowired랑 같은 역활을 한다. 다만 EntityManager를 받아올 경우에는 @PersistenceContext를 적용시켜야 한다 그 이유는 무엇일까?

1. EntityManager를 사용할 때 주의해야 할 점은 **여러 쓰레드가 동시에 접근하면 동시성 문제가 발생하여 쓰레드 간에는 EntityManager를 공유해서는 안됩니다.**
	- Spring Bean은 싱글톤 이기 때문에 모든 쓰레드가 공유합니다.
	- 하지만 @PersistenceContext를 사용하면 동시성 문제를 예방할 수 있습니다.
2. 동시성이 발생하지 않는 이유
	-  스프링 컨테이너가 초기화되면서 @PersistenceContext으로 주입받은 EntityManager를 Proxy로 감쌉니다.
	- 그리고 EntityManager 호출 시 마다 Proxy를 통해 EntityManager를 생성하여 Thread-Safe를 보장합니다.
```gpt
@PersistenceContext`는 Spring에서 JPA(EntityManager)를 주입하기 위해 사용되는 어노테이션입니다. 이 어노테이션을 사용하면 Spring은 컨테이너가 초기화될 때 `EntityManager`의 인스턴스를 프록시로 감쌀 것입니다. 

프록시는 실제 `EntityManager`를 감싸고 있으며, 이 프록시를 통해 `EntityManager`에 액세스됩니다. 이렇게 함으로써 스프링은 각각의 스레드에 대해 단일 `EntityManager` 인스턴스를 유지할 수 있으며, 이는 스레드 안전성을 보장합니다.

일반적으로 JPA에서 `EntityManager`는 스레드 간에 공유되어서는 안 됩니다. 따라서 Spring은 스레드당 하나의 `EntityManager` 인스턴스를 제공하여 이러한 문제를 방지합니다. 이를 통해 개발자가 여러 스레드에서 `EntityManager` 인스턴스를 공유하고 있더라도 JPA 작업이 안전하게 이뤄질 수 있습니다.

즉, `@PersistenceContext`를 사용하면 스프링이 `EntityManager`를 프록시로 감싸고 이를 통해 각 스레드에 대해 새로운 `EntityManager` 인스턴스를 생성하여 스레드 안전성을 보장합니다.
```

삭제를 하는 작업을 보면 id를 통해 실질적인 데이터를 객체로 받아오고 해당 객체를 삭제하도록 요청하는 부분을 볼 수 있다. 

JPA의 동작을 SQL로 보고싶다면 
```properties
spring.jpa.show-sql=true
```
application.properties 파일에 다음 행을 추가하자.


# Spring Data JPA
세상에나.. JPA에서 만든 CouresJPARepository class를 그냥 만들어준다.. 구현이 필요없다.
```JAVA
public interface CourseSpringDataJpaRepository extends JpaRepository<Course, Long> {  
  
}
```

Data Entity Course와 id의 값만 넣어주면 끝이다..

CourseSpringDataJpaRepository 객체의 메소드를 살펴보면 여러기능들이 있다. 하지만 내가 원하는 커스텀 기능이 있을 수 있다. 예를들면 이름으로 데이터를 가져오고 싶다면 어떻게 해야할까?

```JAVA
public interface CourseSpringDataJpaRepository extends JpaRepository<Course, Long> {  
  
    List<Course> findByAuthor(String author);  
    List<Course> findByName(String name);  
}
```
???????????????????????????? 어이가 없다 이게 끝이다. 왜지??????????

# JDBC vs Spring JDBC vs JPA vs Spring JPA
- ### JDBC
	- SQL작성이 많이 필요함.
	- Java코드도 많이 필요로 함
- ### Spring JDBC
	- SQL작성이 많이 필요함.
	- Java코드는 적게 요구함
- ## JPA
	- query에 대해 걱정할 필요가 없음
	- 엔티티를 Table에 맵핑하기만 하면 됨.
- ## Spring JPA
	- JPA를 더 심플하게 사용가능하다.
	- 많은것을 알아서 해줌.


### Hibernate vs JPA
- ### JPA
	- JPA 는 기술명세, API
- ### Hibernate
	- JPA의 구현체
항상 JPA를 사용하자! 다른구현체로 바꾼다고 하여도 문제가 생기지 않는다!