리스트를 순회를하려고한다. 전통적인 방법은 for문을 사용하는 것이다.
```JAVA
public class FP01 {  
    public static void main(String[] args) {  
        printAllNumbersInListStructured(List.of(12,9,13,4,6,2,4,12,15));  
    }  
  
    private static void printAllNumbersInListStructured(List<Integer> numbers) {  
        for(int number : numbers) {  
            System.out.println(number);  
        }  
    }  
}
```

이를 Functional하게 해보자.
```JAVA
//        for(int number : numbers) {  
//            System.out.println(number);  
//        }  
        numbers.stream()  
                .forEach(System.out::println);
```
stream() 을 이용하여 리스트의 각 요소를 하나씩 순회하는 하나의 흐름을 만들어준다.
이후 forEach()를 이용하여 각 요소들이 실행할 것을 정의해준다.
forEach내부에는 static 메소드의 레퍼런스를 넣어준다.


짝수만 출력해주고 싶다면 어떻게 해야할까?
```JAVA
for(int number:numbers) {  
    if(number % 2 ==0)  
        System.out.println(number);  
}
```
이것이 구조적으로 평소에 해오던 방식이다. 이를 함수형으로 바꿔보겠다.
```JAVA
private static boolean isEven(int number) {  
    return number % 2 == 0;  
}
...
numbers.stream()  
        .filter(FP01::isEven)  
        .forEach(System.out::println);
```
filter메소드는 스트림에서 filter메소드의 인자로 받는 static메소드를 통과하는 인자들의 새로운 흐름을 만들어낸다. 그리고 그 흐름에서 forEach를 해준다.

하지만 static메소드를 추가로 만들어서 이용하는것은 알아보기도 어려울 뿐더러 복잡하게 느껴질 수 있다. 이를 람다식으로 개선해 보겠다.
```JAVA
        numbers.stream()  
//                .filter(FP01::isEven)  
                .filter(number->number%2 == 0)  
                .forEach(System.out::println);
```
훨씬 간단해진 것을 알 수 있다.

각 요소들이 실행한 결과를 하나의 스트림으로 받고싶을 경우엔 어떻게 해야할까?
map을 사용하면 된다.
```JAVA
numbers.stream().map(number->number * number).forEach(System.out::println);
```
numbers 리스트를 스트림으로 만들고 이를 제곱한 새로운 스트림을 forEach로 출력해주었다. map에서 리턴하는 값으로 새로운 실행 흐름이 만들어 지는 것이다.
```JAVA
list.stream().map((number)-> {  
    System.out.println(number);  
    return number * number;  
}).forEach(System.out::println);
```
다음과 같이 실행할 경우 number 출력 이후 number * number 가 출력된다. 처음에는 모든 number들이 출력되고 그 이후 모든 number * number 값이 출력될 것이라고 예상하였지만 다르게 동작한다.
실행결과
![](https://i.imgur.com/tL3Ddv8.png)


## Prediciate
```JAVA
@FunctionalInterface  
public interface Predicate<T> {  
  
    /**  
     * Evaluates this predicate on the given argument.     *     * @param t the input argument  
     * @return {@code true} if the input argument matches the predicate,  
     * otherwise {@code false}  
     */    boolean test(T t);
```

Predicate는 함수형 프로그래밍에서 if의 역할을 해주는 인터페이스로 테스트를 실행하는 함수를 저장해두고 이를 실행하여 true 혹은 false를 반환해준다. 

## Optional
Java에서 가장 골치 아픈 문제 중 하나가 null pointer exception이다 이는 널을 참조할때 생기는 문제인데 이를 해결하기 위한 방안으로 Optional이 있다.
```JAVA
Predicate<? super String> predicate = course->course.startsWith("Z");  
Optional<String> StartWithZCourse = courses  
        .stream()  
        .filter(predicate)  
        .findFirst();
```
다음과 같은 코드가 있다 이는 Z로 시작하는 문자열을 찾아서 반환하는데 만약 Z로 시작하는 문자열이 없다면 문제가 일어날 것이다. 때문에 Optional객체를 반환하도록 설계되었다.

Optional객체는 **isEmpty()** 메소드와 **isPresent()** 메소드가 있는데 이를 통해 반환값이 비워져 있는지 확인 할 수 있다.

**get()** 메소드로는 안에 들어있는 값을 가져올 수 있지만 만약 비워져 있는 상태에서 가져오려 시도할 경우 **null pointer exception**이 발생할 것이다.


```JAVA
<? super T>
```
처음보는 표현이라서 찾아 보았다. 이는 T혹은 T의 상위 클래스라는 의미이다.

제네릭 타입 매개변수 안에 들어가는 표현을 찾아보았다.
1. **T**: "Type"의 약자로, 제네릭 클래스나 메서드에서 사용되는 임의의 타입을 나타냅니다. 일반적으로 가장 많이 사용되는 제네릭 타입 매개변수입니다.
    
2. **U**: "Type"의 다른 표현으로, 또 다른 제네릭 타입 매개변수를 나타냅니다. 주로 T와 함께 사용되어 다중 제네릭 매개변수를 표현할 때 사용됩니다.
    
3. **V**: "Value"의 약자로, 제네릭 클래스나 메서드에서 추가적인 값 타입을 나타냅니다. 보통 U와 함께 사용되어 두 가지 이상의 타입을 표현할 때 사용됩니다.
    
4. **S**: "Subtype"의 약자로, 상속 관계에서 하위 타입을 나타냅니다. 주로 제네릭 타입 매개변수로서, 상위 클래스 혹은 인터페이스의 하위 클래스를 나타내기 위해 사용됩니다.
    
5. **E**: "Element"의 약자로, 주로 컬렉션에서 요소의 타입을 나타냅니다. 예를 들어, ArrayList<E>에서 E는 리스트의 요소 타입을 나타냅니다.
