# Swagger fox + Spring Boot Actuator = boom! 


평화로운 따스한 오후

    // swagger fox
    implementation 'io.springfox:springfox-boot-starter:3.0.0'
    implementation("io.swagger:swagger-annotations:1.5.21")
    implementation("io.swagger:swagger-models:1.5.21")
    
    //Spring Boot Actuator
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    runtimeOnly 'io.micrometer:micrometer-registry-prometheus'

개발 중인 프로젝트에 메트릭 정보를 prometheus로 얻고 싶은 마음에

Swagger-fox를 사용중이던 프로젝트에 Spring Boot Actuator 디펜던시 불러주었을 때

> Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException:

그는 나에게로 와서 NPE 가 되었다.

[https://github.com/springfox/springfox/issues/3791](https://github.com/springfox/springfox/issues/3791)

 [Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException in 2.5.0-SNAPSHOT ·

Please take the time to search the repository, if your question has already been asked or answered. What version of the library are you using? Is it the latest version? The latest released version ...

github.com](https://github.com/springfox/springfox/issues/3791)

Swagger는 모든 end point에 대해 노출하는 작업을 하고

Actuator 은 정해진 end point를 생성하여 노출시키는 작업을 하니 이 둘이 충돌을 일으키고 있던 것

![](29575931-0a5b-4997-9b85-a1e251822ce7.png)

나보다 먼저 NPE 선물을 받은 선조들의 지혜를 따르자면

spring boot 버전을 2.5.x로 내려라!

swagger doc으로 변경해라!

가 빠른 해결 방법인 것 같았다.

버전을 내리기에는 자존심이 허락하지 않으며

swagger doc으로 변경하는 일은 너무 배보다 배꼽이 큰 느낌이니 다른 방식으로 해결하자

    spring:
      mvc:
        pathmatch:
           matching-strategy: ant_path_matcher

먼저 application.yml 에서 위의 내용을 추가하자

swagger fox 버전을 내리라는 것과 영향이 있는 부분인데 

swagger fox는 SpringMVC 가 ant\_path\_matcher을 사용하여 경로를 찾는다는 가정하에 동작하는데

SpringMVC가 2.6 버전 이후 PathPattern\_based matcher로 변경하면서 문제가 발생한 것.

이후 SpringMVC의 URL 경로와 Actuator의 end point를 관리 가능하게 하는 매핑 클래스를 빈으로 등록한다.

    @Bean
    public WebMvcEndpointHandlerMapping webEndpointServletHandlerMapping(
      WebEndpointsSupplier webEndpointsSupplier
      , ServletEndpointsSupplier servletEndpointsSupplier
      , ControllerEndpointsSupplier controllerEndpointsSupplier
      , EndpointMediaTypes endpointMediaTypes
      , CorsEndpointProperties corsProperties
      , WebEndpointProperties webEndpointProperties
      , Environment environment) {
      
        List<ExposableEndpoint<?>> allEndpoints = new ArrayList();
        Collection<ExposableWebEndpoint> webEndpoints = webEndpointsSupplier.getEndpoints();
        allEndpoints.addAll(webEndpoints);
        allEndpoints.addAll(servletEndpointsSupplier.getEndpoints());
        allEndpoints.addAll(controllerEndpointsSupplier.getEndpoints());
        String basePath = webEndpointProperties.getBasePath();
        EndpointMapping endpointMapping = new EndpointMapping(basePath);
        boolean shouldRegisterLinksMapping = this.shouldRegisterLinksMapping(webEndpointProperties, environment, basePath);
        return new WebMvcEndpointHandlerMapping(endpointMapping, webEndpoints, endpointMediaTypes,
            corsProperties.toCorsConfiguration()
            , new EndpointLinksResolver(allEndpoints, basePath), shouldRegisterLinksMapping, null);
    }
    
    
    private boolean shouldRegisterLinksMapping(WebEndpointProperties webEndpointProperties, Environment environment,
      String basePath) {
      
        return webEndpointProperties.getDiscovery().isEnabled() && (StringUtils.hasText(basePath) || ManagementPortType.get(
            environment).equals(ManagementPortType.DIFFERENT));
    }

이렇게 작업해 주면 swagger fox와 Actuator 가 동작하게 된다.

이 작업을 하게 된 이유가 Prometheus를 연결하기 위함 이였기에

연결을 해보면

![](5b7a51d4-f246-470a-8c77-1234673f1504.png)

> "INVALID" "\\"" is not a valid start token

짜잔?

![](6f265fc1-1fb5-4a34-a2ca-8b2e126d85b6.png)

젠장2

Actuator 이 정상 동작하여 메트릭 정보가 나오지만

![](9b8c99b1-0e65-4821-8d97-b2d804f987aa.png)

개행 역활을 수행하는 이스케이프 문자인 '\\n' 가 그대로 노출되고 있다.

Json으로 표현이 안되고 String으로 표현되고 있는 이 상태는 HttpMessageConverter의 문제였으며

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    
        StringHttpMessageConverter stringHttpMessageConverter = new StringHttpMessageConverter();
        stringHttpMessageConverter.setDefaultCharset(StandardCharsets.UTF_8);
        converters.add(stringHttpMessageConverter);
    
        GsonHttpMessageConverter gsonHttpMessageConverter = new GsonHttpMessageConverter();
        gsonHttpMessageConverter.setGson(gson);
        converters.add(gsonHttpMessageConverter);
    }

직렬화 가능하도록 Converter을 설정하여 해결하였다.

참고로 만약 위의 상황에서 Swagger-fox 가 아닌 Swagger-doc였다면 swagger 웹 화면에서 아래와 같이 나왔을 것이다.

> Unable to render this definition  
> The provided definition does not specify a valid version field.  
> Please indicate a valid Swagger or OpenAPI version field. Supported version fields are swagger: "2.0" and those that match openapi: 3.0.n (for example, openapi: 3.0.0).

![](00a92164-d18d-4703-b57c-e75abce346e5.png)

위의 오랜 작업을 통해 드디어 Swagger-fox를 사용하는 SpringBoot 프로젝트에 Actuator를 추가하여 Prometheus에서 메트릭 정보를 수집할 수 있게 되었다.

![](6e7670f4-4597-48fe-bf87-21a4a865fc1e.png)

...

.....

.......

그냥 전체 검색의 도움을 받아 Swagger-fox 를 Swagger-doc으로  마이그레이션 했다..

![](d47956e6-6ad5-4985-b750-f38155f0b0c1.png)

 **swagger-fox 어노테이션**

 **swagger-doc 어노테이션**

 @Api

 @Tag

 @ApiOperation

 @Operation

 @ApiModelProperty

 @Schema

 @ApiImplicitParam, @ApiParam

 @Parameter

 @ApiImplicitParams

 @Parameters

 @ApiIgnore

 @Hidden

참고 및 출처

[https://github.com/springfox/springfox/issues/3791](https://github.com/springfox/springfox/issues/3791)

[https://oingdaddy.tistory.com/411](https://oingdaddy.tistory.com/411)

[https://stackoverflow.com/questions/60044853/openapi-escapes-json-response-in-spring-boot-application](https://stackoverflow.com/questions/60044853/openapi-escapes-json-response-in-spring-boot-application)