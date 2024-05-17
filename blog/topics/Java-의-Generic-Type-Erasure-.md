# Java 의 Generic Type Erasure 


![](2b354353-6936-4a9a-b5ca-ceac424672c6.png)

아니 object 야

### Type Erasure 

타입 소거, 타입 삭제 등 다양하게 불리는 Type Erasure는

제네릭의 원소 타입을 컴파일 타입에서만 검사하고 런타임에서는 해당 타입 정보를 교체하는 것을 의미한다.

교체를 아무렇게나 하는 게 아니라

unbiunded type 은 object로 

즉 <?> , <T>와 같은 형태는 object로 교체하고

bounded type 은 bound type로

즉 <T extends String>는 String로 교체한다.

### unbiunded type의 Type Erasure 

    public class CommonClass {
    
      public static String method(String str) { return str; }
    }
    
    @Test
    public void commonClassTest() throws NoSuchMethodException {
    
      Class<?> type = CommonClass.class
          .getMethod("method", String.class)
          .getReturnType();
    
     System.out.println("type = " + type.getName());
    }
    
    
    > type = java.lang.String

위의 코드를 보았을 때

return type 가 string라는 것은 쉽게 파악할 수 있다.

하지만 제네릭을 사용하는 경우 Type Erasure 가 발생하여 예상과 다른 결과를 얻게 된다.

    public class UnBoundedClass {
    
      public static <T> T method(T t) { return t;}
    }
    
    @Test
    public void unBoundedClassTest() throws NoSuchMethodException {
    
      Class<?> type = UnBoundedClass.class
          .getMethod("method", String.class)
          .getReturnType();
    
      System.out.println("type = " + type.getName());
    }

위와 같은 코드를 작성한다면 어떻게 될까?

리플랙션을 사용하여 메서드를 호출하고

String class를 전달하였으니

return type 가 String 일 것으로 예상이 된다.

하지만 결과는

> java.lang.NoSuchMethodException: com.typeerasure.UnBoundedClass.method(java.lang.String)

String을 파라미터로 받는 mehtod 메서드가 없다는 에러를 만나게 된다.

위에서 알아봤던 것처럼 unbiunded type 은 object로 변경되었기에 발생한 일이다.

    public class UnBoundedClass {
    
      public static Object method(Object t) { return t; }
    }

즉 런타임에서는 위와 같이 변경이 되었다는 것이다. 따라서

    @Test
    @DisplayName("UnBounded type 은 object 가 된다")
    public void unBoundedClassTest() throws NoSuchMethodException {
    
      Class<?> type = UnBoundedClass.class
          .getMethod("method", Object.class)
          .getReturnType();
    
      System.out.println("type = " + type.getName());
    }
    
    > type = java.lang.Object

Object를 매개변수로 전달하면 정상 동작하는 것을 볼 수 있다.

하지만 코드에서 사용할 때는

제네릭으로 지정된 타입을 리턴 받아 사용해야 하는데

object로 반환이 되는 것이었다면

object를 지정한 타입으로 변환하는 다운캐스팅을 해야 한다.

이러한 과정은 컴파일 시 다운캐스팅 하는 코드가 자동으로 추가가 되어있기에 당연하게 동작하고 있던 것.

    String oTos = (String)method();

### bounded type의 Type Erasure 

    public class BoundedClass {
    
      public static <T extends String> T method(T t) { return t; }
    }
    
    @Test
    @DisplayName("Bounded type 은 Bound 가 된다")
    public void boundedClassTest() throws NoSuchMethodException {
    
      Class<?> type = BoundedClass.class
          .getMethod("method", String.class)
          .getReturnType();
    
      System.out.println("type = " + type.getName());
    }
    
    > type = java.lang.String

bounded type을 사용하는 경우를 보면

type의 제한을 걸어두었기 때문에 bound type로 변환된 모습을 볼 수 있다.

이러한 Type Erasure로 인해 아래와 같은 일도 가능하게 된다.

    @Test
    @DisplayName("String list 에 Integer 추가 하기")
    public void test() {
      List<String> strings = new ArrayList<>();
    
      strings.add("first String");
      addInteger(strings, 1000);
      System.out.println("strings = " + strings);
    }
    private void addInteger(List strings, Integer i) {
      strings.add(i);
    }
    
    > strings = [first String, 1000]

런타임에서는 List <String>과 List <Interer> 은 둘 다 List객체일 뿐이다.

자바는 사실 동적 타이핑 언어가 아니었을까?

### 이런 걸 왜 만든 거야

제네릭은 자바의 등장과 함께 나온 것이 아닌

자바 5 버전에서 등장하였다.

그렇기에 이전 버전의 호환성을 위해 선택한 방법이 Type Erasure이다.

이를 통해 호환성과 타입별 새로운 클래스를 생성하지 않아도 된다는 이점으로

제네릭이 런타임 오버헤드를 발생하지 않게 되었지만

컴파일에서 오류를 잡지 못하는 문제가 생기게 되었다.

참고 및 출처

[https://docs.oracle.com/javase/tutorial/java/generics/unboundedWildcards.html](https://docs.oracle.com/javase/tutorial/java/generics/unboundedWildcards.html)

[https://www.david-merrick.com/2017/09/19/what-is-type-erasure/](https://www.david-merrick.com/2017/09/19/what-is-type-erasure/)