

## Spring container == Spring context == IOC(제어의 역전)  container

- Spring Bean과 생명주기를 관리
- 여러 클래스들과 configuration => container  의 인풋
- container의 아웃풋 => Ready System

## Ready System
- 모든 Bean을 관리하는 Spring context가 존재

## 1. Bean Factory
- Basic Spring Container

## 2. Application Context
- Advanced Spring Container with enterprise-spcific features
	- easy to use web applications
	- easy internationaliztion
	- easy integraion with Spring AOP

## Which one to use? 
- Most enterpise applications use Application Context
	- Recommended for web applications, web services - REST API and microservices

## Bean Factory VS Application Context
- Bean Factory는 Lazy-Loading 방식
	- 빈을 사용할때 빈을 로딩한다.(가벼운 경량 컨테이너)
- Application Context는 Eager-loading 방식
	- 빈을 미리 로딩시킨다.
	- Bean Factory를 상속받았기 때문에 Lazy-Loading도 가능하다.
- MessageSource를 이용한 국제화 가능(?)
- EnvironmentCapable 환경 변수를 이용한 로컬,개발, 운영 구분(?)
- ApplicationEventPublisher 애플리케이션 이벤트를 이용하여, 이벤트를 발행하고 구독하는 모델을 편리하게 지원(?)
- ResourceLoader 를 이용하여 편리하게 파일, 클래스패스 등의 리소스를 조회(?)


```
BeanFactory라고 말을 할때는 빈을 생성하고 관계를 설정하는 IoC의 기본 기능에 초점을 맞춘 것이다.

ApplicationContext라고 말을 할때는 별도의 정보를 참고해서 빈의 생성, 관계 설정 등의 제어의 총괄에 초점을 맞춘 것이다.
```

## ApplicationContext에서 Lazy initalized 를하기

^27a32c

- @Lazy 어노테이션을 추가하면 실질적으로 Bean이 불릴때 생성자가 호출된다.
- Lazy initialization을 할 경우 메모리를 절약 가능하나 에러가 발생할 경우 런타임에 발견된다.