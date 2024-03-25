

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