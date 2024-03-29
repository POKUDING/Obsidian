
Spring에서 중요한 Scope 2가지가 있다. 하나는 **싱글톤** 하나는 **프로토타입**이다.

기본적으로는 싱글톤이나 @Scope 어노테이션으로 프로토타입을 만들 수 있다.
```JAVA
@Component  
class NormalClass {  
  
}  
  
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)  
@Component  
class ProtoTypeClass {  
  
}
```

먼저 두개의 다른 Class를 정의하여 주었다. 둘의 차이는 무엇일까?


```JAVA
System.out.println(context.getBean(NormalClass.class));  
System.out.println(context.getBean(NormalClass.class));  
  
System.out.println(context.getBean(ProtoTypeClass.class));  
System.out.println(context.getBean(ProtoTypeClass.class));  
System.out.println(context.getBean(ProtoTypeClass.class));
```

각각 여러번의 Class 호출을 진행하였다.


```
com.junhyupa.learnjavaspring.examples.e1.NormalClass@3315d2d7
com.junhyupa.learnjavaspring.examples.e1.NormalClass@3315d2d7
com.junhyupa.learnjavaspring.examples.e1.ProtoTypeClass@d6e7bab
com.junhyupa.learnjavaspring.examples.e1.ProtoTypeClass@5fa07e12
com.junhyupa.learnjavaspring.examples.e1.ProtoTypeClass@55b53d44

종료 코드 0(으)로 완료된 프로세스
```

결과창을 보면 NormalClass의 경우에는 여러번 호출하여도 같은 주소값을 가지는 것을 확인할 수 있다.
반면, ProtoTypeClass의 경우에는 매번 호출할 때 마다 다른 주소값을 가지는 것을 확인할 수 있다.
싱글톤의 경우 하나의 인스턴스를 만들어서 해당 클래스가 필요로 할 때 같은 인스턴스를 주입하여 실행시켜주는 방식이다.

프로토타입의 경우에는 필요할때마다 인스턴스를 만들어서 실행시켜준다.


```JAVA
@Component  
class NormalClass {  
    public NormalClass() {  
        System.out.println("make Normal class");  
    }  
  
}  
  
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)  
@Component  
class ProtoTypeClass {  
    public ProtoTypeClass() {  
        System.out.println("make ProtoType class");  
    }  
  
}
```

각각 클래스의 생성자가 호출될때 출력을 하도록 생성자를 만들어 주었다.
```JAVA
System.out.println(context.getBean(ProtoTypeClass.class));  
System.out.println(context.getBean(ProtoTypeClass.class));  
System.out.println(context.getBean(ProtoTypeClass.class));  
System.out.println(context.getBean(NormalClass.class));  
System.out.println(context.getBean(NormalClass.class));
```

또한 각 클래스를 호출하는 순서도 바꾸어 주었다.
```
make Normal class
make ProtoType class
com.junhyupa.learnjavaspring.examples.e1.ProtoTypeClass@3315d2d7
make ProtoType class
com.junhyupa.learnjavaspring.examples.e1.ProtoTypeClass@d6e7bab
make ProtoType class
com.junhyupa.learnjavaspring.examples.e1.ProtoTypeClass@5fa07e12
com.junhyupa.learnjavaspring.examples.e1.NormalClass@55b53d44
com.junhyupa.learnjavaspring.examples.e1.NormalClass@55b53d44
```
출력문을 보면 Singleton Class 인 NormalClass는 프로그램이 실행되고 먼저 생성이되고 ProtoTypeClass는 매 호출시점에 생성자가 호출되는 것을 알 수 있다.

즉 Spring IoC container하나당 하나의 singleton Class를 가지게 되고 ProtoTypeClass의 경우 여러개의 인스턴스가 있을 수 있다.

그외의 스코프들이 있다.
- Request - HTTP요청 하나당 하나의 인스턴스를 갖게 된다.
- Session - 세션 하나당 하나의 인스턴스를 갖게 된다.
- Application - 하나의 App당 하나만 존재한다.
- Websocket - 하나의 웹소켓 당 하나의 인스턴스를 갖게된다.

## Java Singleton (GOF) VS Spring Singleton
- Spring Singleton: IoC container 하나당 하나의 인스턴스
- Java Singleton: JVM 하나에서 인스턴스 하나
- 99.99%의 경우 JVM에 여러개의 IoC container를 실행하지 않기 때문에 같다고 볼 수 있다.