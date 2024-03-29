## Spring Framework에 대한 이해
- Spring Framework 를 Web Backend Framework로 이해하고 있었던 것에 대한 오류를 정정했다.
- Spring Framework 는 Enterprise Application 을 만들기 위한 프레임워크로  대규모 Application을 효율적이고 안정적이게 만드는 것에 의의를 두고 만들어진 프레임워크이다.

## Spring Framework의 특징
- IoC(제어의 역전): 객체의 생성과 소멸 등의 객체의 관리를 중앙집중형으로 프레임워크가 한다. 다른객체를 사용하고자 할때는 IoC container가 주입해주는 의존성으로 연결된다.
- DI(의존성 주입): 객체간 느슨한 결합을 위하여 객체에서 다른 객체를 필요로 할 경우에는 의존성을 주입받아서 사용한다. 코드의 재사용성을 증가시킬 수 있으며 자원을 효율적으로 사용할 수 있다. 또한 메모리 낭비등을 방지할 수 있다.
- AoP(관점지향프로그래밍): ... https://goddaehee.tistory.com/154