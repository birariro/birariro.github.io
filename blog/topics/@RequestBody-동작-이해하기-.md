# @RequestBody 동작 이해하기 


API 의 Body 를 통해 JSON 데이터를 받기위해 @RequestBody 를 사용하여 받는다.  
  
이는 JSON,XML 과 같은 HTTP 데이터 형식을 JSON 으로 변환한다.  
스프링 내부에서 HttpMessageConverter을 통해 알맞는 오브젝트로 변환된다.  
  
일단 간단한 API를 작성해서 확인해보자

    @PostMapping("/request-body")
    public String requestBodyTest(@RequestBody RequestBodyTestDto requestBodyTestDto){
        return requestBodyTestDto.getParamString();
    }
    

    public class RequestBodyTestDto{
    
        private String name;
        private int age;
        private boolean die;
    
        public String getParamString(){
            return name + ", " + age + ", "+ die;
        }
    }
    

위와같이 Controller 와 DTO 를 준비를 하면  
API는 호출이 되지만

    null, 0, false
    

Body 안에는 아무값이 들어있지않다.

Getter / Setter 추가
------------------

    @Getter
    public class RequestBodyTestDto{
    	//생략
    }
    

getter 을 추가하여 똑같이 실행해보면

    kogo, 29, false
    

정상적으로 값이 들어오는 모습을 볼수있으며

    @Setter
    public class RequestBodyTestDto{
    	//생략
    }
    

setter 을 추가해도

    kogo, 29, false
    

정상적으로 받는 모습을 볼수있다.  
즉 RequestBody 를 통해 JSON 을 파싱하는경우  
Getter, Setter 중 한개는 필요하지만 그렇다고 이 메소드를 완전히 사용해서 파싱하고있지않다.

파싱 원리
=====

> The body of the request is passed through an \[HttpMessageConverter\](<[https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html)\>)  to resolve the method argument depending on the content type of the request.

    AbstractJackson2HttpMessageConverter.class
    
    InputStream inputStream = StreamUtils.nonClosing(inputMessage.getBody());
    if (inputMessage instanceof MappingJacksonInputMessage) {
    	Class<?> deserializationView = ((MappingJacksonInputMessage) inputMessage).getDeserializationView();
    	if (deserializationView != null) {
    		ObjectReader objectReader = objectMapper.readerWithView(deserializationView).forType(javaType);
    		if (isUnicode) {
    			return objectReader.readValue(inputStream);
    		}
    		else {
    			Reader reader = new InputStreamReader(inputStream, charset);
    			return objectReader.readValue(reader);
    		}
    	}
    }
    

파싱을 하기위해 HttpMessageConverter 를 사용하고있으며 ObjectMapper 을 이용하여 역직렬화를 통해 Json 을 JAVA 객체로 생성한다.  
이때 JSON → JAVA객체로 변환하기위해 ObjectMapper은  
Getter,Setter 에서 앞의 Get,Set 를 지우고 나머지 문자의 첫문자를 대문자로 변경하여

    getName -> name
    

이 문자를 참조하여 변환한다.

**ETC**
=======

직렬화면 기본 생성자가 필요한거 아닌가?
----------------------

리플렉션 관련 자료를 보면 기본생성자를 통해 객체를 생성하기에 반드시 필요하다는걸 알수있다.  
그렇다면 왜 기본생성자가 반드시 필요할까?  
리플렉션 는 이미 로딩이 완료된 클래스에서 다른 클래스를 동적으로 로딩하여 생성자,맴버필드,맴버 메소드를 사용할수있게 하는 자바 API 이지만  
리플렉션이 얻지못하는것이 생성자의 파라미터이다.  
따라서 기본생성자 없이는 객체를 생성 할수 없기에  
기본 생성자로 객체를 생성하고 필드 정보를 얻어와서 값을 할당하는것이 리플렉션이다.

    @Setter
    public class RequestBodyTestDto{
    
        private String name;
        private int age;
        private boolean die;
    
        public RequestBodyTestDto(String name) {
            this.name = name;
        }
        public String getParamString(){
            return name + ", " + age + ", "+ die;
        }
    }
    

만약 위와같이 파라미터가있는 별도의 생성자를 만들어 두는경우  
기본 생성자를 만들지 않으면 리플렉션이 실패한다.

    JSON parse error: Cannot construct instance of ...
    

변수명에 is,get,set 이 들어가면?
-----------------------

위에서 말한것처럼 객체를 파싱하기위해 get/set 메소드를 사용하여 참조할 값을 추적을 하게되는데  
이는 is 도 같다.

    @Setter
    public class RequestBodyTestDto{
    
        private String getName;
        private int setAge;
        private boolean isDie;
    }
    

만약 위와같이 변수명을 만들게되면  
getter 은

    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public boolean isDie() {
        return isDie;
    }
    

이러한 모양이 될것이고 ObjectMapper 는 값을 추적하기위해  
isDie() 를 die 로 바꿔서 die를 참조할텐데  
실제 필드는 die 이 아니라 isDie이기에 실패한다.

**결론**
======

*   RequestBody 는 JSON,XML 등을 JAVA 객체로 변환하는 어노테이션이다.
*   변환하기위해 ObjectMapper 를 사용한다.
*   ObjectMapper 는 변경할 객체안에있는 get/set 메소드의 이름을 변경하는것으로 참조할 값을 추적한다.
*   Dto는 반드시 getter, setter 둘중 하나가 있어야한다.
*   별도의 생성자가 있는경우 기본생성자가 반드시 있어야한다.
*   그냥 @NoArgsConstructor, @Getter 넣고 시작하자.