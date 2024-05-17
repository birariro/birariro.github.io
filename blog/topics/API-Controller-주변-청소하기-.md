# API Controller 주변 청소하기 


![](1ec919c1-b1eb-4fa8-97f7-fc7d160de3e7.png)

spring boot를 사용하여 api 개발을 하면서 swagger을 함께 사용하다 보면 

많은 API들이 다들 위와 같은 모습을 하고 있는 게 뭐랄까.. 애정이 안 간다.

코드를 아래의 기준으로 정리를 하여 애정이 생길만한 이쁜 모습으로 바꿔보려 한다.

*   한눈에 보기 좋을 것
*   중복 코드가 없을 것
*   api 파라미터는 스네이크 케이스이고 controller 인자는 카멜 케이스 일 것

![](e5f9d7a6-d1f9-4c82-9131-265b2eea0a13.png)

### @ApiOmplicitParams 만 사용하기

    @ApiOperation(value = "주문 조회")
    @ApiImplicitParams({
      @ApiImplicitParam(name = "productId", value = "제품 id", 
          required = false, dataType = "string", paramType = "query"),
      @ApiImplicitParam(name = "paymentType", value = "결제 수단", 
          required = false, dataType = "string", paramType = "query"),
      @ApiImplicitParam(name = "page", value = "페이지 위치", 
          required = true, dataType = "int", paramType = "query", defaultValue = "0"),
      @ApiImplicitParam(name = "size", value = "크기", 
          required = true, dataType = "int", paramType = "query", defaultValue = "10"),
      @ApiImplicitParam(name = "sort", value = "정렬 대상", 
          required = false, dataType = "string", paramType = "query"),
      @ApiImplicitParam(name = "order", value = "정렬 방법", 
          required = false,  dataType = "string", paramType = "query")
    
    })
      @GetMapping("/order/v1")
      public ResponseEntity getOrdersV1(
           String productId, String paymentType, int page, int size, 
           String sort, String order) {
    
        return ResponseEntity.ok().build();
      }

controller의 method 인자가 단순해졌기 때문에

그나마 보기 좋아졌지만

여전히 파라미터의 정보에 대한 코드가 길다

또한 api 파라미터에서 받는 변수 그대로 메서드 인자로 들어가기 때문에

\[api 파라미터는 스네이크 케이스이고 controller 인자는 카멜 케이스 일 것\]에 부합하지 못한다

대신 Swagger UI에서 각 파라미터를 상세하게 설명할 수 있다.

### @RequestParam 만 사용하기

    @ApiOperation(value = "주문 조회")
    @GetMapping("/order/v2")
    public ResponseEntity getOrdersV2(
          @RequestParam(value = "product_id",required = false) String productId
        , @RequestParam(value = "payment_type", required = false) String paymentType
        , @RequestParam(value = "page") int page
        , @RequestParam(value = "size") int size
        , @RequestParam(value = "sort", required = false) String sort
        , @RequestParam(value = "order", required = false) String order) {
    
      return ResponseEntity.ok().build();
    }

@ApiImplicitParam을 사용했을 때에 비해서는

api 파라미터와 메서드 인자가 한눈에 보기 좋아 가독성도 좋아진 것 같다

    @ApiOperation(value = "주문 조회")
    @GetMapping("/order/v2")
    public ResponseEntity getOrdersV2(
          @RequestParam(value = "product_id",required = false) String productId
        , @RequestParam(value = "payment_type", required = false) String paymentType
        , @RequestParam(value = "page") int page
        , @RequestParam(value = "size") int size
        , @RequestParam(value = "sort", required = false) String sort
        , @RequestParam(value = "order", required = false) String order) {
    
      return ResponseEntity.ok().build();
    }
    
    @ApiOperation(value = "주문 조회 비슷한 api")
    @GetMapping("/order/v2/api")
    public ResponseEntity getOrdersV2api(
          @RequestParam(value = "product_name",required = false) String productName
        , @RequestParam(value = "page") int page
        , @RequestParam(value = "size") int size
        , @RequestParam(value = "sort", required = false) String sort
        , @RequestParam(value = "order", required = false) String order) {
    
      return ResponseEntity.ok().build();
    }

API 가 많아질수록 비슷한 파라미터를 받는 API들이 생기고 결국  \[중복 코드 제거\]에 부합하지 못한다

또한 Swagger UI에서 각 파라미터 설명을 추가할 수 없다.

### DTO로 사용하기

위와 같이 중복 파라미터 부분을 DTO 로 분리해서 사용할 수 있다.

    @Getter 
    @Setter 
    public class PageRequest  {
    
        @ApiModelProperty(value = "페이지 위치", example = "0", required = true)
        private int page = 0;
        @ApiModelProperty(value = "크기", example = "10", required = true)
        private int size = 10;
        @ApiModelProperty(value = "정렬 대상")
        private String sort;
        @ApiModelProperty(value = "정렬 방법")
        private String order;
    }
    
    @Getter 
    @Setter 
    public class OrderPageRequest extends PageRequest {
    
        @ApiModelProperty(value = "결제수단")
        private String paymentType;
        @ApiModelProperty(value = "제품 Id")
        private String productId;
    }

위와 같이 페이지 관련 기능을 PageRequest로 분리하였고

API의 고유한 파라미터를 다른 DTO로 만들어서 상속받아 사용하도록 한다.

    @ApiOperation(value = "주문 조회")
    @GetMapping("/order/v3")
    public ResponseEntity getOrdersV3(OrderPageRequest orderPageRequest) {
    	return ResponseEntity.ok().build();
    }

이렇게만 두면 얘만 어노테이션이 없어서 허전하니

기능은 하지 않지만 명시를 위한 어노테이션을 생성해 주었다.

    @Target(ElementType.PARAMETER)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface QueryParameter {
    }
    
    @ApiOperation(value = "주문 조회")
    @GetMapping("/order/v3")
    public ResponseEntity getOrdersV3(@QueryParameter OrderPageRequest orderPageRequest) {
    	return ResponseEntity.ok().build();
    }

이렇게 되면 setter에 의해 api 파라미터와 dto가 연결되기에 

api 파라미터를 카멜케이스로 사용해야 한다.

"api 파라미터는 스네이크 케이스이고 controller 인자는 카멜 케이스로 가능할 것"을 지키기 위해서는 

api 파라미터에 맞는 스네이크 케이스 setter을 제공하고

카멜 케이스 변수랑 연결한다.

    @Getter
    public class OrderPageRequest extends PageRequest {
    
        @ApiModelProperty(value = "결제수단", name = "payment_type")
        private String paymentType;
        @ApiModelProperty(value = "제품 Id", name = "product_id")
        private String productId;
    
        public void setPayment_type(String paymentType) {
        	this.paymentType = paymentType;
        }
    
        public void setProduct_id(String productId) {
            this.productId = productId;
        }
    }

이렇게 해서 API 끼리 중복되는 파라미터는 상속을 통해서 처리할 수 있게 되었다.

대신 API 마다 DTO 가 늘어나고

파라미터를 확인하려면 DTO를 매번 찾아가서 확인해야 한다.

### 중복만 dto로 만들기 + @RequestParam 사용하기

    @ApiOperation(value = "주문 조회")
    @GetMapping("/order/v4")
    public ResponseEntity getOrdersV4(
      @RequestParam(value = "product_id",required = false) String productId
    , @RequestParam(value = "payment_type", required = false) String paymentType
    , @QueryParameter PageRequest PageRequest) {
    	return ResponseEntity.ok().build();
    }

자주 쓰는 부분은 DTO로 처리하고

API의 특징적인 파라미터는 @RequestParam로

많은 방법 중 이 방법이 마음에 들기에 이 방식으로 하기로 땅땅땅

> 엥 이건 Swagger-fox 인데 Swagger-doc 면 어떻게 하라는 거임?  
>   

    @Operation(summary = "주문 조회")
    @GetMapping("/order/v4")
    public ResponseEntity getOrdersV4(
      @Parameter(description = "product_id",required = false) String productId
    , @Parameter(description = "payment_type", required = false) String paymentType
    , @ParameterObject PageRequest PageRequest) {
    	return ResponseEntity.ok().build();
    }
    
    
    @Getter 
    @Setter 
    public class PageRequest  {
    
        @Schema(description = "페이지 위치", example = "0", required = true)
        private int page = 0;
        @Schema(description = "크기", example = "10", required = true)
        private int size = 10;
        @Schema(description = "정렬 대상")
        private String sort;
        @Schema(description = "정렬 방법")
        private String order;
    }

어노테이션만 바꾸면 되는걸..