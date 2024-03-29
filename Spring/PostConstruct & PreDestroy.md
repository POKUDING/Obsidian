## @PostConstruct
- **@PostConstruct**어노테이션은 생성자가 실행된 직후(모든 의존성이 주입된 직후) 실행시킬 메소드에 붙여주는 어노테이션이다.

## @PreDestroy
- **@Predestroy**어노테이션은 객체가 소멸하기 직전에 실행시킬 메소드에 붙여주는 어노테이션이다.

```JAVA
@Component  
class SomeClass {  
    private SomeDependency someDependency;  
  
    public SomeClass(SomeDependency someDependency) {  
        super();  
        this.someDependency = someDependency;  
        System.out.println("All Dependencies are ready!");  
    }  
  
    @PostConstruct  
    public void initialize() {  
        someDependency.getReady();  
    }  
  
    @PreDestroy  
    public void cleanup() {  
        System.out.println("Clean Up!");  
    }  
}  
  
@Component  
class SomeDependency {  
  
    public void getReady() {  
        System.out.println("Some logic using someDependency");  
    }  
  
}  
@Configuration  
@ComponentScan("com.junhyupa.learnjavaspring.examples.f1")  
public class PrePostAnnotationsContextLauncherApplication {  
  
     public static void main(String[] args) {  
        try (var context =  
                     new AnnotationConfigApplicationContext  
                    (PrePostAnnotationsContextLauncherApplication.class)) {  
            System.out.println("main logic");  
        }  
  
    }  
}
```


실행 결과
```
All Dependencies are ready!
Some logic using someDependency
main logic
Clean Up!
```
