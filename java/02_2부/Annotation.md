# Annotation

자바 소스 코드에 추가하여 사용할 수 있는 메타데이터의 일종이다. jdk 1.5에 추가되었다.

## 애노테이션의 종류

### 표준 애노테이션

자바에서 기본으로 제공해주는 애노테이션입니다.  
ex) `@Override`, `@Deprecated`, `@SuppressWarnings`, `@FunctionalInterface` 등이 있습니다.

### 메타 애노테이션

애노테이션의 메타 정보를 다루는 애노테이션입니다.

### 주요 메타 애노테이션

#### @Retention

자바 컴파일러가 애노테이션을 언제까지 영향을 미쳐야 하는지를 서술합니다.

- `RetentionPolicy.SOURCE` : 컴파일 전까지만 유효 (컴파일 이후 삭제), 클래스 파일에 존재 x
- `RetentionPolicy.CLASS` : 컴파일러가 클래스를 참조할 때까지 유효 즉, 컴파일 시에만 필요하고 런타임에는 필요하지 않은 경우에 사용됩니다.
- `RetentionPolicy.RUNTIME` : 컴파일 이후에도 JVM에 의해 계속 참조가 가능하다.

#### @Target

애노테이션을 적용할 수 있는 자바 요소의 종류를 지정합니다.

- `ElementType.PACKAGE` : 패키지 선언에 애노테이션을 적용합니다.

```java
@MyPackageAnnotation
package com.example.myapp;
```

- `ElementType.TYPE` : 클래스, 인터페이스(애노테이션 타입), 열거형에 적용됩니다.

```java
@MyTypeAnnotation
public class MyClass { ... }
```

- `ElementType.ANNOTATION_TYPE` : 다른 애노테이션에 적용 할 수 있습니다.

```java
@MyAnnotationAnnotation
public @interface MyAnnotation { ... }
```

- `ElementType.CONSTRUCTOR` : 생성자 선언에 적용됩니다.

```java
public class MyClass {
    @MyConstructorAnnotation
    public MyClass() { ... }
}
```

- `ElementType.FIELD` : 클래스의 멤버 변수에 적용됩니다.

```java
public class MyClass {
    @MyFieldAnnotation
    private int myField;
}
```

- `ElementType.LOCAL_VARIABLE` : 로컬 변수에 적용됩니다.

```java
public void myMethod() {
    @MyLocalVariableAnnotation int localVariable;
}

```

- `ElementType.METHOD` : 메서드 선언에 적용됩니다.

```java
public class MyClass {
    @MyMethodAnnotation
    public void myMethod() { ... }
}
```

- `ElementType.PARAMETER` : 메서드의 매개변수에 적용됩니다.

```java
public void myMethod(@MyParameterAnnotation int param) { ... }
```

- `ElementType.TYPE_PARAMETER` : 타입 매개변수(제너릭)에 적용됩니다.

```java
public class MyClass<@MyTypeParameterAnnotation T> { ... }
```

- `ElementType.TYPE_USE` : 타입이 사용될 수 있는 모든 곳에 적용됩니다.

```java
public @MyTypeUseAnnotation List<String> myList;
```

## 애노테이션의 규칙

- 타입 : 기본형, String, enum, 애노테이션, 클래스만 허용된다.
- 애노테이션 메서드에는 매개변수 사용을 금지한다.
- 예외 선언이 불가능하다.
- 타입 매개변수 사용이 불가능하다. ex) `<T>`

## 동작원리

- 컴파일러의 애노테이션 스캔 : Java 컴파일러가 소스 파일을 스캔하여 애노테이션을 찾고, 애노테이션들이 타겟으로 설정됩니다.
- 컴파일러는 각 애노테이션에 대해 등록된 애노테이션 프로세서를 호출합니다.
- 애노테이션 프로세서가 애노테이션이 적용된 요소를 분석하고 필요한 처리를 수행합니다.
