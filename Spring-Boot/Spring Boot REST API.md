**@RestContoller** 어노테이션으로 해당 클래스를 RestController로 Bean에 등록할 수 있다.

**@RequstMapping** 을 이용하여 요청을 받았을때 해당 메소드를 실행 할 수 있도록 지정 가능하다.
**@RequestMapping(value="" ,metohd= "")**
value = "/root" value 는 요청받을 위치를 넣을 수 있다.
method="" RequestMethod.GET .POST .DELETE ... 해당 메소드가 처리할 요청 메소드를 정의 할 수 있다.


JAVA Bean([[JAVA Bean VS POJO VS Spring Bean#^803ea8]]) 을 만들어서 요청으로 내보내면 클라이언트는 JSON으로 받을 수 있다.

```JAVA
//JAVA Bean
public class HelloWorldBean {  
  
    private String message;  
  
    public HelloWorldBean(String message) {  
        this.message = message;  
    }  
  
    public String getMessage() {  
        return message;  
    }  
  
    public void setMessage(String message) {  
        this.message = message;  
    }  
  
    @Override  
    public String toString() {  
        return "HelloWorldBean{" +  
                "message='" + message + '\'' +  
                '}';  
    }  
}
```

```JAVA
//contorller
@GetMapping("/hello-world-bean")  
public HelloWorldBean helloWorldBean() {  
    return new HelloWorldBean("Hello World");  
}
```

```JSON
{"message":"Hello World"}
```



요청은 어떻게 처리될까?
- 제일먼저 distpathcerServlet으로 처리된다.
	- 어떤 url과 상관없이 제일 먼저 dispatcherServlet으로 요청이 들어오고 dispatcherServlet이 해당 url과 일치하는 컨트롤러에 매핑해준다.
	- ![](https://i.imgur.com/gHEg3ol.png)
	- 로그들을 보면 GET "/hello-world-bean"으로 요청이 들어오고 나서 부터의 처리가 쭉 나온다. 처음에 DispatcherServlet이 이 요청을 받은 로그를 볼 수 있다.
	- DispatcherServlet은 어떻게 관리될까?
	- Spring Boot 에서 DispatcherServletAutoConfiguration 을 통하여 자동으로 설정된다.

HelloWorldBean이 어떻게 JSON으로 변환되었을까?
	- **@ResponseBody** + JacksonHttpMessageConverters 가 Bean class를 자동으로 JSON으로 변환해준다.
	- **@ResponseBody** 어노테이션은 **@RestController** 어노테이션에 포함되어있다.
	- JacksonHttpMessageConverters는 Spring Boot 에서**JacksonHttpConvertersConfiguration** 을 통하여 자동으로 설정된다.

누가 에러를 처리하나?
	- **ErrorMvcAutoConfiguration** 에 의해 기본적으로 관리된다.

tomcat, Spring, Spring MVC, Jackson 등이 어떻게 다 사용가능한가?
- 이것이 Spring Boot의 마법이다. 모든것들을 먼저 설정하여 사용할 수 있게 해준다.
- spring-boot-starter-web 을 통하여 들어가 보면 모든 의존성이 설정되어있는것을 확인 할 수 있다.


Path Parameters
- ex) /user/{id}/todos/{id} => /user/2/todos/200
```JAVA
  @GetMapping("/hello-world/{path}")  
public HelloWorldBean helloWorldPathVariable (@PathVariable String path) {  
    return new HelloWorldBean(path);  
}
```
위와 같이 {} 로 묶은 변수로 PathParmeters를 받을 수 있다. 주의 해야할점은 요청주소의 변수이름과 메소드의 매개변수가 이름이 같아야 한다.
![](https://i.imgur.com/F0jkqZ7.png)
({path} 로 들어온 "hello-hello" 가 변수로 들어가서 처리되었다.)



### Post 요청 응답에 location추가
Post요청을 받게되면 해당 내용을 생성한 이후 생성된 위치를 알려주는것이 좋다.
이를 위해서는 헤더에 location을 추가해야한다.
```JAVA
import java.net.URI;
...
@PostMapping("/users")  
public ResponseEntity<User> createUser(@RequestBody User user) {  
    User savedUser = userDaoService.save(user);  
    URI location = ServletUriComponentsBuilder  
            .fromCurrentRequest()  
            .path("/{id}")  
            .buildAndExpand(savedUser.getId())  
            .toUri();  
    return ResponseEntity.created(location).build();  
}
```
URI class는 위치를 저장하는 클래스이다. ServletUriComponentsBuilder 메소드를 이용하여 URI를 생성하는데 .fromCurrentRequest()를 이용하여 현재 요청받은 위치를 우선 두고 path(/{id}) 를 이용하여 뒤에 변수를 붙여 주었다. 그뒤 저장된 데이터의 id를 넣어주면 해당 변수에 id가 대입된다. 이후 toUri()메소드로 빌드하면 된다.

ResponseEntity 는 응답에 대한 form을 받아주는데 .created()를 이용하면 201 응답이 전달되고 인자로는 URI를 받도록 되어있다.


### 404에러 핸들링
현재는 /user/{없는 유저 id} 로 GET요청을 할 경우 
```JAVA
return users.stream().filter(predicate).findFirst().get();
```
위 코드에서 에러가 발생하여 클라이언트에게 500에러를 전송하게 된다. 이는 .get()메소드가 널을 참조하려고 하는 null pointer exception으로 JAVA에서 흔하게 보이는 문제이다. 이를 해결하고 404 에러를 전송해 보자.
```JAVA
//.get()의 구현
public T get() {  
    if (value == null) {  
        throw new NoSuchElementException("No value present");  
    }  
    return value;  
}
```




> 수정한 코드
```JAVA
return users.stream().filter(predicate).findFirst().orElse(null);
```

우선 .get() 메소드를 .orElse()메소드로 변경해 주었다. orElse()메소드를 보면
```JAVA
public T orElse(T other) {  
    return value != null ? value : other;  
}
```
다음과 같이 구현되어 있는데 Value가 null이 아니면 Value를 리턴하고
Value가 null이면 입력받은 인자를 리턴한다.

우린 null을 넣어주었으니 null을 리턴할 것이다.

이후 null을 핸들링 해주지 않으면 유저는 빈화면을 보게되는데 이를 막기 위해 null 핸들링 로직을 추가한다.
```JAVA
if (user == null)  
    throw new UserNotFoundException("id :" + id);
```
user 가 null일경우 UserNotFoundExeption을 throw 해준다.

UserNotFoundException은 커스텀 Exception으로 다음과 같이 구현해 주었다.
```JAVA
@ResponseStatus(code = HttpStatus.NOT_FOUND)  
public class UserNotFoundException extends RuntimeException {  
    public UserNotFoundException(String message) {  
        super(message);  
    }  
}
```
**@ResponseStatus** 어노테이션을 이용하여 예외가 발생하였을 때 던질 오류를 정의할 수 있다. code = httpStatus.NOT_FOUND를 통하여 없는 id에 접근할 경우 404코드를 응답한다.