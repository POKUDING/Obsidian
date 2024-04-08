SpringBoot  이전에는 어떻게 개발을 했을까?

- 의존성관리
- 각종 설정
- Bean 관리
- 로깅, Error Hanling, Monitoring
등등을 직접 해줘야 했다. 하지만 spring boot 에서는 이것들에게서 자유롭게 해준다.


#### 시작
시작은 start.spring.io 에서 세팅을 해주면 편하다. 의존성에는 Spring web을 추가해주었다.

#### Spring Boot Starter
- 편리한 의존성 디스크립터

스프링 앱을 만들기 위해서는 여러 개의 configuration 이 필요하다
- Component Scan, DispatcherServelt,Data Sources. JSON Conversion...
이것들을 간단하게 할 수  없을까?
- Auto Configuration: 당신의 앱을 위한 자동설정
	- 다음에 결정됨
		- 클래스 경로가 어떤 프레임워크에 있는가?
		- 기존에 지정된 설정(어노테이션 등)이 있는가?
Example: Spring Boot Starter Web
- Dispatcher Servlet (DispatcherServletAutoConfiguration)
- Embedded Servlet Container - Tomcat is th default (EmveddedWebServerFactoryCusomizerAutoConfiguration)
- Default Error Pages (ErrorMvcAutoConfiguration)
- Beac<->JSON (JacksonHttpMessageConvertersConfiguration)

### Spring Boot DevTools 를 이용하여 빠르게 빌드하기
- 개발자의 생산성 향상
- 코드가 수정될때마다 재시작하지 않고 바로 적용시킬 수 없을까?

pom.xml
```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-devtools</artifactId>  
</dependency>
```

### log의 레벨로 더 많은 로그 보는 방법
```
\\application.properties
logging.level.org.springframework=debug
```
debug , trace, info, warning, error, off등의 옵션이 있다.
각 다른 프로필의 설정은 application-**profilename**.properties 로 설정해주면된다. 그리고 기본 application.porperties안에
```
spring.profiles.active=profilename
```
를 입력해주면 해당 프로필의 설정이 적용된다.

같은 코드도 여러 환경에서 실행시킬 것이다.
이 환경들을 각각의 프로필로 만들어서 관리할 수 있다.
- dev
- qa
- stage
- prod

### ConfigurationProperties
현재 프로필에 따라 다른 속성값을 가지게 만들 수 있다.
예를들어 dev와 prod의 db url이 바뀌어야 한다면 하나하나 바꿔주는것 보다 프로필로 관리하면 편해질 것이다.
```
//application-properties
currency-service.url=http://default.in28minutes.com  
currency-service.username=default1username  
currency-service.key=defaultkey
```
application-properties안에 다음과 같이 url, username, key를 정의하고 이를 가져와서 쓰고자 한다.

우리는 @Componet와 @ConfigurationProperties를 이용하여 이 문제를 해결할 수 있다.
```JAVA
@ConfigurationProperties(prefix = "currency-service")  
@Component  
public class CurrencyServiceConfiguration {  
    private String url;  
    private String username;  
    private String key;  
  
    public String getUrl() {  
        return url;  
    }  
  
    public void setUrl(String url) {  
        this.url = url;  
    }  
  
    public String getUsername() {  
        return username;  
    }  
  
    public void setUsername(String username) {  
        this.username = username;  
    }  
  
    public String getKey() {  
        return key;  
    }  
  
    public void setKey(String key) {  
        this.key = key;  
    }  
}
```
Getter 와 Setter를 만들어주어야 한다. 

Controller로 현재의 프로퍼티를 가져와보자
```JAVA
@RestController  
public class CurrencyConfigurationController {  
  
    @Autowired  
    private CurrencyServiceConfiguration currencyServiceConfiguration;  
  
    @RequestMapping("/currency-configuration")  
    public CurrencyServiceConfiguration retrieveAllCourses() {  
        return currencyServiceConfiguration;  
    }  
}
```
@Autowired를 이용하여 현재의 properties를 주입 받아서 사용 가능하다.

marven을 사용할 경우 mvn clean install 을 실행하고 생성된 jar 파일을 터미널에서
```bash
JAVA -jar ...jar
```
로 실행하면 된다.


#### Monitor Application using Spring Boot Actuator
모니터링 및 관리가능
beans 엔드포인트
health 
metrics
mappings
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-actuator</artifactId>  
</dependency>
```
```properties
management.endpoints.web.exposure.include=*
```
다음과 같이 설정한 이후 /actuator 주소로 접근하면 여러 상태와 정보를 볼 수 있다. \*로 모두 노출시킬 경우 모니터링을 위한 정보수집이 광범위하게 일어남으로써 메모리와 CPU가 낭비된다. 필요한 부분만 노출시키자.


### Spring Boot VS Spring MVC vs Spring
- Spring Framework: Dependency Injection
	- @Component, @Autowired, @ComponenScan ... etc..