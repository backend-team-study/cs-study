# 01. Java 전반부

> 2024.01.14 일

- 자료형
  - call by value vs call by reference
  - generic
  - wrapper class
  - String & StringBuffer & StringBuilder
  - equals(), hashcode()
  - final
- jvm
- garbage collection
- 접근 제한자
- 클래스, 생성자
  - inner class
  - java reflection
  - collection framework
- 객체지향
  - 추상화
  - 캡슐화
  - 상속
  - 다형성
  - 추상클래스와 인터페이스



## 질문 모음

- 불변 객체가 무엇인지 설명하고 대표적인 Java의 예시를 설명해주세요.
- 참조 타입일 경우 추가적인 작업은 어떤게 있는지 설명해주세요
- 불변 객체나 final을 굳이 사용해야 하는 이유가 있을까요?

- Call by Reference와 Call by Value 의 차이

- 제네릭(Generic)
  - 처리 방법
- Wrapper class
  - Boxing/unboxing

- String & StringBuffer & StringBuilder

- equals()와 hashcode()에 대해 설명해 주세요.
  - 본인이 hashcode() 를 정의해야 한다면, 어떤 점을 염두에 두고 구현할 것 같으세요?
  - 그렇다면 equals() 를 재정의 해야 할 때, 어떤 점을 염두에 두어야 하는지 설명해 주세요.
- new String()과 리터럴("")의 차이에 대해 설명해주세요.

- final 키워드를 사용하면, 어떤 이점이 있나요?

  - final 키워드 (final/finally/finalize)

  - 그렇다면 컴파일 과정에서, final 키워드는 다르게 취급되나요?

- JVM이 정확히 무엇이고, 어떤 기능을 하는지 설명해 주세요.
  - 그럼, 자바 말고 다른 언어는 JVM 위에 올릴 수 없나요? 
  - 반대로 JVM 계열 언어를 일반적으로 컴파일해서 사용할 순 없나요? 
  - VM을 사용함으로써 얻을 수 있는 장점과 단점에 대해 설명해 주세요.
  - JVM과 내부에서 실행되고 있는 프로그램은 부모 프로세스 - 자식 프로세스 관계를 갖고 있다고 봐도 무방한가요?
- 자바가 컴파일 되는 과정을 설명해주세요
- Java의 GC에 대해 설명해 주세요.
  - finalize() 를 수동으로 호출하는 것은 왜 문제가 될 수 있을까요? 
  - 어떤 변수의 값이 null이 되었다면, 이 값은 GC가 될 가능성이 있을까요?

- static class와 static method를 비교해 주세요.
  - static 을 사용하면 어떤 이점을 얻을 수 있나요? 어떤 제약이 걸릴까요? 
  - 컴파일 과정에서 static 이 어떻게 처리되는지 설명해 주세요.
- non-static 멤버와 static 멤버의 차이

- 클래스와 객체에 대해 설명해주세요.
  - 클래스 멤버 변수 초기화 순서
- 생성자(Constructor)에 대해 설명해주세요.
- Inner Class(내부 클래스)의 장점에 대해 설명해주세요.
- 리플렉션(Reflection)이란 무엇인지 설명해주세요.
  - 리플렉션은 어떤 경우에 사용되는지 설명해주세요.
  - 의미만 들어보면 리플렉션은 보안적인 문제가 있을 가능성이 있어보이는데, 실제로 그렇게 생각하시나요? 만약 그렇다면, 어떻게 방지할 수 있을까요?
  - 리플렉션을 언제 활용할 수 있을까요?

- 인터페이스와 추상클래스의 차이 (Interface VS Abstract Class)
  - interface와 abstract에 대해서 설명하세요.
  - • 왜 클래스는 단일 상속만 가능한데, 인터페이스는 2개 이상 구현이 가능할까요?
- 다형성이란?
  - Overriding vs Overloading
- 인터페이스와 추상 클래스의 차이에 대해 설명해 주세요.
  - 왜 클래스는 단일 상속만 가능한데, 인터페이스는 2개 이상 구현이 가능할까요?

- SOLID원칙에 대해서 설명하세요
- 객체지향언어가 절차지향언어와 다르게 가지는 장점은 무엇인가요?