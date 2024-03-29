
## @Qualifier
- 여러 후보의 Bean이 호출될 자격이 있을 때, 기본으로 선택되는 Bean

## @Primary
- 연결하기 위한 빈을 특정

## 어떤 것을 사용해야 할까?
- 의존하는 class의 관점에서 결정해야 한다.
- @Autowired => 기본으로 설정된 Bean을 가져다 줘
- @Autowired + @Qualifiler => Qualifier가 일치하는 Bean을 가져다 줘
- @Qualifier가 설정되어 있지 않은 빈을 가져오고 싶을 경우
	- @Autowired + @Qulifier("Bean name") 으로 실질적인 Bean의 이름을 넣으면 특정이 가능하다.