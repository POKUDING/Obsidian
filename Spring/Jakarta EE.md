**J2EE -> Java EE -> Jakarta EE 로 발전**
**J2EE**
- Java 2 platform Enterprise Edition
**Java EE**
- Java Platform Enterprise Edition(Rebranding)
**Jakarta EE**
- Oracle이 Java EE의 권한을 eclips에 넘김
- Specifications
	- Jakrta Server Page(JSP)
	- Jakrta Standard Tag Library(JSTL)
	- Jakrta Enterprise Beans(EJB)
	- Jakrta RESTful Web Services(JAX-RS)
	- Jakarta Bean Validation
	- Jakarta Context and Dependency injextion(CDI)
	- Jakarta Persitence(JPA)
- Supported by Spring 6 and Spring Boot 3
	- 우리가 javax가 아닌 jakarta package를 사용하는 이유

#### Jakarta Contexts & Dependency injextion(CDI)
-  @Component 를 대신하여 @Named 를 사용할 수 있다.
-  @Autowired 대신 @Inject 를 사용할 수 있다.

```JAVA
@Named  
class BusinessService {  
    DataService dataService;  
  
    public DataService getDataService() {  
        return dataService;  
    }  
  
//    @Autowired  
    @Inject  
    public void setDataService(DataService dataService) {  
        this.dataService = dataService;  
        System.out.println("setterinjection");  
    }  
}
```

