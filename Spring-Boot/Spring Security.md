- 인증을 쉽게 구현할 수 있게 해주는 프레임워크

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-security</artifactId>  
</dependency>
```
pom.xml에 다음과 같이 spring-boot-starter-security 패키지를 의존성에 추가해주면 Sprin Security가 적용된다.

적용 후 어떠한 위치로 접근해도 /login으로 redirection을 하며 로그인을 요구한다. spring을 부팅하면 비밀번호가 나오는데 id에 user를 pw에 콘솔에 나온 pw를 넣어주면 로그인이 가능하다.

하지만 매번 재실행 할때마다 비밀번호를 콘솔에서 찾아서 가져오는건 불편하다.

```JAVA
package com.junhyupa.springboot.myfirstwebapp.security;  
  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.security.core.userdetails.User;  
import org.springframework.security.core.userdetails.UserDetails;  
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;  
import org.springframework.security.crypto.password.PasswordEncoder;  
import org.springframework.security.provisioning.InMemoryUserDetailsManager;  
  
import java.util.function.Function;  
  
@Configuration  
public class SpringSecurityConfiguration {  
    @Bean  
    public InMemoryUserDetailsManager createUserDetailsManager() {  
        Function<String, String> passwordEncoder  
                = input -> passwordEncoder().encode(input);  
  
        UserDetails userDetails = User.builder()  
                .passwordEncoder(passwordEncoder)  
                .username("junhyupa")  
                .password("dummy")  
                .roles("USER","ADMIN")  
                .build();  
  
        return new InMemoryUserDetailsManager(userDetails);  
    }  
  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
}
```
#의문점 
위와 같이 설정해주면.. 된다....? ㅎㅎ;

```JAVA
@Bean  
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
    http.authorizeHttpRequests(  
            auth-> auth.anyRequest().authenticated());  
    http.formLogin(withDefaults());  
    http.csrf().disable();  
    http.headers().frameOptions().disable();  
    return http.build();  
}
```
이건 또 뭘까? ㅎㅎ