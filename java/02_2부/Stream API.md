# Stream API

JDK 1.8부터 Steramp API와 람다식, 함수형 인터페이스 등을 지원하면서 Java를 통해 함수형 프로그래밍이 가능하게 되었다. 그 중에서 Stream API는 데이터를 추상화하고, 처리하는데 자주 사용되는 함수들의 모음이다.

### Stream API의 세 가지 단계

Stream은 데이터를 처리하기 위해 다양한 연산들을 지원한다. Stream이 제공하는 연산을 이용하면 복잡한 작업들을 간단히 처리할 수 있는데, 스트림에 대한 연산은 크게 **생성, 중간 연산, 최종 연산**로 나눌 수 있다.

### 중간 연산의 특징

Stream은 지연 평가 전략 (Lazy Evaludation)을 수행하는데, 간단히 말하면 결과값이 필요할 때까지 계산을 늦추는 기법이다. 다만, 이러한 방식으로 코드가 동작하기 위해 내부적으로 준비 과정이 필요하므로 무조건 효율적인 방법이라고 보기는 어렵지만, 준비 과정을 통해 스트림 파이프라인이 어떠한 중간 연산과 최종 연산으로 구성되어있는지에 대해 검사하고 그 결과를 바탕으로 JVM이 최적화를 진행하여 연산을 수행하게 된다.

중간 연산의 최적화 전략으로 루프 퓨전과 쇼트서킷이 사용되는데, 루프 퓨전은 연속적으로 체이닝된 복수의 스트림 연산을 하나의 연산 과정으로 병합시키는 것을 말한다. 쇼트서킷은 불필요한 연산을 의도적으로 수행하지 않음으로써 실행 속도를 높이는 기법을 말한다. (and && 연산과 같은 것)

```java
class Data { 
		private final int value; 

		public Data(int value) { 
				this.value = value; 
		} 

		public void runOperationA() { 
				System.out.printf("데이터 %d의 작업A%n", value); 
		} 

		public void runOperationB() { 
				System.out.printf("데이터 %d의 작업B%n", value); 
		} 
		
		public void runOperationC() { 
				System.out.printf("데이터 %d의 작업C%n", value); 
		} 
	
		public int getValue() { return value; } 
		
		@Override 
		public String toString() { return "Data{" + value + '}'; } 
}

// Stream
Stream.of(new Data(1), new Data(2), new Data(3)) 
					.peek(Data::runOperationA) // 1
					.peek(Data::runOperationB)  // 2
					.forEach(Data::runOperationC); // 3

// for 문
List<Data> datas = List.of(new Data(1), new Data(2), new Data(3));
for (Data data : datas) {
		data.runOperationA();
		data.runOperationB();
		data.runOpertaionC();
}

// 출력
// 데이터 1의 작업A 
// 데이터 1의 작업B 
// 데이터 1의 작업C 
// 데이터 2의 작업A 
// 데이터 2의 작업B 
// 데이터 2의 작업C 
// 데이터 3의 작업A 
// 데이터 3의 작업B 
// 데이터 3의 작업C
```

위 예시와 같은 클래스가 있을 때 Stream 을 사용하면 체이닝된 순서대로 호출하는 것이 아니라 아래의 for 문처럼 동작하여 개별 스트림 요소에 접근하는 횟수를 최소화 시킨다는 것이다.

### 실행 순서에 대한 고려

루프 퓨전으로 인해 실행 순서에 대한 고려가 필요할 수도 있는데, 아래의 예시를 통해 알아보자.

```java
Stream.of("a", "b", "c", "d", "e")
        .map(s -> {
            System.out.println("map: " + s);
            return s.toUpperCase();
        })
        .filter(s -> {
            System.out.println("filter: " + s);
            return s.startsWith("A");
        })
        .forEach(s -> System.out.println("forEach: " + s));
        
/*
map: a
filter: A
forEach: A <- 한 번만 동작했음
map: b
filter: B
map: c
filter: C
map: d
filter: D
map: e
filter: E
*/
```

쉽게 예상할 수 있듯이 중간 연산 과정인 map()과 filter()는 모든 문자열에 대해 각각 다섯 번 호출되었고, 최종 연산인 forEach()는 한 번만 호출되었다.

```java
Stream.of("a", "b", "c", "d", "e")
        .filter(s -> {
            System.out.println("filter: " + s);
            return s.startsWith("a");
        })
        .map(s -> {
            System.out.println("map: " + s);
            return s.toUpperCase();
        })
        .forEach(s -> System.out.println("forEach: " + s));

/*
filter: a
map: a
forEach: A
filter: b
filter: c
filter: d
filter: e
```

하지만, 위와 같이 수정한다면 filter() 만 다섯 번 호출되고 map과 forEache는 각각 한 번 씩만 호출되는 것을 볼 수 있다. 때문에 Stream API를 사용할 때는 반드시 연산 순서를 고려하여 최적화를 진행해야 한다.

### 병렬 스트림

Stream은 아주 많은 양의 데이터를 처리해야 하는 경우에 런타임 성능을 높이기 위해 병렬 스트림을 제공하고있다. ForkJoinPool.commonPool()을 사용하며, 내재되어 있는 TrheadPool의 기본 값은 5개이다.

병렬 스트림으로 생성하기 위한 parallelStream() 메서드와 중간 연산을 병렬로 처리하기 위한 parallel() 메서드를 제공한다.

> `-Djava.util.concurrent.ForkJoinPool.common.parallelism=5` JVM 매개변수로 별도 지정이 가능하다.

### peek() vs forEach()

peek() 의 경우 중간 연산 과정으로 Stream 의 요소들을 대상으로 Stream에 영향을 주지 않고 특정 연산을 수행하기 위해 사용한다.

forEach()의 경우 최종 연산 과정으로 실제 요소들에 영향을 줄 수 있으며, 반환값이 존재하지 않는다.