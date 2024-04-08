**@Entity** 어노테이션을 붙이면 해당 class는 데이터베이스 table과 매칭해준다.

**@Column(name="")**어노테이션을 이용하여 매칭될 column을 바꿔줄 수 있다.

**userName**과 같이 중간에 대문자가 들어가면 **USER_NAME**으로 자동으로 바꿔준다.

기본적으로 data.sql이 Entity보다 먼저 실행된다. 테이블이 생성되기 전에 data.sql이 실행되면 오류가 발생한다 이러한 경우
```properties
spring.jpa.defer-datasource-initialization=true
```
다음과 같이 application.properties 파일에 추가해주면 된다.

그래도 오류가 나서 보니 @Entity어노테이션에 이름을 지정해줄 수 있다. (ex: @Entity(name="todoabc))
연습으로 이름을 설정해 놓아서 테이블 이름이 매칭되지 않아서 오류가 발생했었다.