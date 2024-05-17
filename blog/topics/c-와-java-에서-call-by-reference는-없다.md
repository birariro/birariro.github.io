# c 와 java 에서 call by reference는 없다


![](731477b4-4789-47ae-9a10-2b9ee8470494.png)

Call by Reference(Pass by Reference)
------------------------------------

call by reference는 프로그래밍 언어 설계에서 사용되는 용어로, 함수에 인수를 전달할 때 값의 복사본 대신 원래 값의 참조를 전달 받는 것을 의미한다.

함수 내에서 참조를 통해 직접 값을 접근함으로써 원본을 변경할 수 있게 된다.

예를 들어 다음과 같은 Java 코드를 살펴보면

    public static void main(String[] args) {
        List list = new ArrayList();
    
        add(list);
        System.out.println(list.size()); //output: 1
    }
    public static void add(List collection){
        collection.add("1");
    }


위 코드는 call by reference 로 보일 수 있지만 사실은 call by value 이다.

C 언어의 창시자인 데니스 리치의 "The C Programming Language"에는 다음과 같이 설명되어 있다

> C에서 호출할 때 매개변수는 그 값만 전달되므로(call by value) 호출된 함수에서..(생략)
>
>
> 함수를 호출할때 변수명을 넘겨주는것이 아니라 값만 넘겨주기 때문이다. 변수명을 넘겨주는것을 call by refenence 라고 하고 값을 넘겨주는것을 call by value 라고 부른다.. (생략)
>
>
> C에서는 호출된 함수가 변수의 값을 변화 시킬수 없다.

java 언어의 창시자인 제임스 고슬링의 "The Java Programming Language"에는 다음과 같이 설명되어 있다.

> 일부 사람들은 객체가 "참조(reference)에 의해 전달된다"고 잘못 말할 것입니다.  
> 프로그래밍 언어 설계에서 "참조에 의한 전달(pass by reference)"이라는 용어는 올바르게 말하자면 함수에 인수가 전달될 때 호출된 함수가 원래 값에 대한 참조를 얻는다는 것을 의미합니다.  
> 그것의 값의 복사본이 아닙니다. 함수가 매개변수를 수정하면 호출 코드의 값이 변경됩니다. 왜냐하면 인수와 매개변수가 메모리의 동일한 슬롯을 사용하기 때문입니다.  
> Java 프로그래밍 언어는 객체를 참조(reference)에 의해 전달하지 않습니다. 참조(reference)를 값으로 전달합니다. 동일한 참조의 두 개의 복사본이 실제 객체를 참조하기 때문에 한 참조 변수를 통해 만든 변경 사항은 다른 변수를 통해 볼 수 있습니다.  
> 매개변수 전달 방법은 값으로 전달 방법 딱 한 가지 입니다 그리고 그것은 모든 것을 단순하게 유지하는 데 도움이 됩니다

따라서, 객체 참조가 실제로는 값으로 전달되고, 두 변수가 동일한 참조를 가리키게 되어 한 변수를 변경하면 다른 변수도 변경된다.

즉, 참조가 전달되는 것이 아니라 참조를 값으로 전달하는 것. (Passes Object references by value)

위 내용을 고려하여, 다시 코드를 살펴보면

    public static void main(String[] args) {
        List list = new ArrayList();
    
        add(list);
        System.out.println(list.size());
    }
    public static void add(List collection){
        collection.add("1");
    }


main 함수의 list 변수는 스택에 할당된 변수이며, 힙 영역을 가리키는 참조 값을 가지고 있다.

add 함수의 인수로 넘겼을 때, add 함수의 매개변수 collection 또한 스택에 할당되며, 전달된 값을 복사하여 사용하게된다

따라서 두 변수는 다른 변수이지만, 같은 메모리 슬롯을 가리키고 있기 때문에 한 변수를 변경하면 다른 변수도 함께 변경된다.