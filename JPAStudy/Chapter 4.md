# 엔티티 매핑
## JPA 매핑 어노테이션 분류
- 객체와 테이블 매핑: **@Entity**,**@Table**
- 기본 키 매핑: **@Id**
- 필드와 컬럼 매핑: **@Column**
- 연관관계 매핑: **@ManyToOne**, **@JoinColumn**
## @ Entity
JPA를 사용해서 테이블과 매핑할 클래스는 **@Entity** 어노테이션이 필수로 붕어야 한다. **@Entity** 어노테이션이 붙은 클래스는 JPA가 관리하는 것으로, 엔티티라 부른다.
### Entity의 속성

| 속성   | 기능                                                                                                   | 기본값                        |
| ---- | ---------------------------------------------------------------------------------------------------- | -------------------------- |
| name | JPA에서 사용할 엔티티 이름을 지정한다. 보통 기본값인 클래스 이름을 사용한다. 만약 다른패키지에 이름이 같은 엔티티 클래스가 있다면 이름을 지정해서 충돌하지 않도록 해야 한다. | 설정하지 않으면 클래스 이름을 그대로 사용한다. |
### @Entity 적용시 주의사항
- 기본 생성자는 필수다(파라미터가 없는 public or protect 생성자).
- final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
- 저장할 필드에 final을 사용하면 안된다.
JPA가 엔티티 객체를 생성할 때 기본 생성자를 사용하므로 이 생성자는 반드시 있어야 한다.

## @Table
**@Table**은 엔티티와 매핑할 테이블을 지정한다. 생략하면 패밍한 엔티티 이름을 테이블 이름으로 사용한다.

| 속성                      | 기능                                                                                                  | 기본값           |
| ----------------------- | --------------------------------------------------------------------------------------------------- | ------------- |
| name                    | 매핑할 테이블 이름                                                                                          | 엔티티 이름을 사용한다. |
| catalog                 | catalog 기능이 있는 데이터베이스에서 catalog를 매핑한다.                                                              |               |
| schema                  | schema 기능이 있는 데이터베이스에서 schema를 매핑한다.                                                                |               |
| uniqueConstraints (DDL) | DDL 생성 시에 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. 참고로 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용된다. |               |
#기술부채 catalog, schema, DDL
> DDL(Data Definition Language)

## 다양한 매핑 사용
```JAVA
@Enumerated(EnumType.STRING) // 1
private RoleType roleType;

@Temproral(TemproralType.TIMESTAMP) // 2
private Date createdDate;

@Temproral(TemproralType.TIMESTAMP)
private Date lastModifiedDate;

@Lob //3
private String description;
```
1. roleType: 자바의 enum을 사용해서 회원의 타입을 구분했다. 이처럼 자바의 enum을 사용하려면 @Enumerated 어노테이션으로 매핑해야 한다.
2. createdDate, lastModifiedDate: 자바의 날짜 타입은 @Temproral을 사용해서 매핑한다.
3. description:  회원을 설명하는 필드는 길이 제한이 없다. 따라서 데이터베이스의 VARCHAR 대신 CLOB 타입으로 지정해야 한다. @LOB을 사용하면 CLOB, BLOB 타입을 매핑할 수 있다.
#기술부채 CLOB, BLOB

## 데이터베이스 스키마 자동 생성
JPA는 데이터베이스 스키마를 자동으로 생성하는 기능을 지원한다. JPA는 매핑정보와 데이터베이스 방언을 사용해서 데이터베이스 스키마를 생성한다.
```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```
persistence.xml 파일을 열어 위의 속성을 추가하면 **애플리케이션 실행 시점에 데이터베이스 테이블을 자동으로 생성**한다.

또한
```xml
<property name="hibernate.show_sql" value="true" />
```
위 속성을 추가하면 테이블생성 DDL(Data Definition Language) 를 출력할 수 있다.

스키마 자동 생성 기능을 사용하면 애플리케이션 실행 시점에 데이터베이스 테이블이 자동으로 생성되므로 개발자가 테이블을 직접 생성하는 수고를 덜 수 있다.
하지만 **스키마 자동 생성 기능이 만든 DDL은 운영 환경에서 사용할 만큼 완벽하지는 않으므로** 개발 환경에서 사용하거나 매핑을 어떻게 해야 하는지 참고하는 정도로만 사용하는 것이 좋다.

#### hibernate.hbm2ddl.auto 속성

| 옵션          | 설명                                                                                                  |
| ----------- | --------------------------------------------------------------------------------------------------- |
| create      | 기존 테이블을 삭제하고 새로 생성한다. DROP + CREATE                                                                 |
| create-drop | create 속성에 추가로 애플리케이션을 종료할 때 생성한 DDL을 제거한다.<br>DROP + CREATE + DROP                                 |
| update      | 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정한다.                                                             |
| validate    | 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다. 이설정은  DDL을 수정하지 않는다.                   |
| none        | 자동 생성 기능을 사용하지 않으려면 hibernate.hbm2ddl.auto 속성 자체를 삭제하거나 유효하지 않은 옵션 값을 주면 된다.(none은 유효하지 않은 옵션 값이다.) |
