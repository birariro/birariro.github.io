# 그놈의 REST 한 API 


HTTP API
========

HTTP API는 서로 정해둔 스펙으로 데이터를 주고받으며 통신하는것을 의미한다.

가장 중요한 특징으로

기능에대해 GET, POST, PUT, DELETE 등 메소드를 통해 여러 행동을 정의하게된다.

회원에 대한 API 를 예로 보면

    http://server.com/users

위와 같은 API 스팩이 있을때

    GET http://server.com/users //전체 유저 조회
    GET http://server.com/users/k4keye //특정 유저 조회
    POST http://server.com/users {id="k4keye"&pwd="pwd"} //특정 유저 생성
    PUT http://server.com/users/k4keye  {id="k4keye"&pwd="new pwd"} //특정 유저 변경
    DELETE http://server.com/users/k4keye //특정 유저 삭제
    DELETE http://server.com/users //서비스 종료

HTTP 메소드를 통해 하나의 자원을 여러 행위로 요청한다.

RES…HTTP API!
=============

> 제발 제약 조건을 따르던지 아니면 REST 말고 다른 단어를 써라

#### 1\. HTTP 의 원칙을 따라 HTTP 메소드를 사용하라  
2\. 일반 텍스트를 반환하지 말고 json 이라는 좋은 포멧을 사용해라  
3\. URI 에 동사를 넣지마 HTTP 메소드가있잖어

#### 4\. 자원명은 복수명사가 좋다더라 (카더라)

아래와 같이 보았을때 user하나를 반환하는지 user 전체를 반환하는지 알수있다고 한다

    GET http://server.com/user
    GET http://server.com/users

#### 5\. 보안상 큰 이슈만 아니라면 본문에 오류 메시지를 반환하여 디버깅을 하는 사람을 도와주라  
6\. 에러 코드 확실하게 줘라

200이라는 성공 응답을 줘놓고 안에서 실패라고 말을 바꾸지 마라

    HTTP/1.1 200 OK
    Content-Type: application/json;
    {
      "statue" : false
      "message" : "파라미터가 부족"
    }

![](5afa8642-729e-4251-8149-ef10464f404a.png)

    HTTP/1.1 400 Bad Request
    Content-Type: application/json;
    {
      "message" : "파라미터가 부족"
    }

#### 7\. 이왕이면 성공 코드도 잘 주자

GET

성공, 응답 데이터 있음(200 ok)

요청성공 이지만 응답 데이터 없을때(204 No Content)

잘못된 리소스 요청(404)

POST

추가 성공, 응답데이터 있음 (201 Created)

추가 성공, 응답데이터 없음 (204 No Content)

잘못된 요청(400 Bad Request)

PUT

수정 성공 (200 or 204)

추가 성공(201 Created)

업데이트 불가(409 충돌 Conflict)

DELETE

삭제 성공 응답 데이터 있음(200 OK)

삭제 성공 응답 데이터 없음(204 No Content)

잘못된 요청(400 Bad Request)

ETC

지원하지 않는 미디어 타입 (415)

인증을 해라(401 Unauthorized)

너의 권한으로는 접근 불가(403 Forbidden)

#### 8\. API URI 마지막에 /(슬레시) 를 넣을지 말지는 확실하게 정해라

    http://server.com/users
    http://server.com/users/

둘중하나는 실패하는 API이다 하지만 이걸 클라이언트는 알기 어렵고Microsoft 의 웹 API 디자인 모범 사례 에는 마지막에 / 를 붙히지 않는다. 서버는 원할한 서비스를 위해 잘못된 요청이오면 정상요청으로 리다이랙트 시켜주는 작업을 해야할것이다.

#### 9\. URL 에는 언더바 말고 하이푼으로 연결해라

    http://server.com/users/nick-name/

#### 10\. URL 로 자원을 식별해야한다.

    GET http://server.com/users/birariro

위와같은 방식은 하면 안된다 REST 는 URL 에 자원이 식별 가능해야하기때문에 

    GET http://server.com/users?id=birariro

파리미터로 넘기는게 아닌 URL 로 표현한다.

REST API
========

HTTP API 와 같이 정해둔 스펙으로 통신하는건 같지만 여러가지 제약조건이 추가된 형태이다.

1.  자원의 식별
2.  메시지를 통한 리소스 조작
3.  자기 서술적 메시지
4.  하이퍼미디어

> 시스템 전체를 통제할수있다고 생각하거나 진화에 관심이 없다면 REST 에대해 따지느라 시간을 낭비하지마라 - 로이필딩  
>   

![](7f3fe105-8dfa-4891-9df1-2c18b18903b7.png)

자기 서술적 메시지
----------

메시지만 보고 해석이 불가능해서 REST 문서가 필요한 경우가 잘못되고있는 대표적인 모습이라 볼수있다.

    GET http://server.com/data
    {
      "myProp": "",
      "myProp3": "test"
    }
    
    PUT http://server.com/data
    
    [
      {
        "op": "replace",
        "path": "/myProp",
        "value": "hi",
      },
      {
        "op": "add",
        "path": "/myProp2/myPropDepth",
        "value": "hello",
      },
      {
        "op": "remove",
        "path": "/myProp3",
      }
    ]
    
    요청 GET http://server.com/data
    결과는??

위와같은 메시지가 왔을때 op, path, value가 무슨 역활을 의미하는지 알기 힘들다.

따라서 이메시지의 의미를 해석할수있게 알려야한다.

    context-type : json/patch+json
    [
      {
        "op": "replace",
        "path": "/myProp",
        "value": "hi",
      },
      {
        "op": "add",
        "path": "/myProp2/myPropDepth",
        "value": "hello",
      },
      {
        "op": "remove",
        "path": "/myProp3",
      }
    ]
    

json/patch 는 PUT통신에서 일부(partial)만을 날려서 수정하는 방식을 의미한다.

→ PATCH Method 써도된다.

*   op : 오퍼레이션
*   path : 대상
*   value : 값

그렇기에 결과를 예상할수있다.

    GET http://server.com/data
    {
      "myProp": "hi",
      "myProp2": {
        "myPropDepth": "hello"
      }
    }

하이퍼미디어 (HATEOAS)
----------------

링크에 사용가능한 URL을 리소스로 전달하여 클라이언트가 참고할수있도록 하는것

이로힌해 서버와 클라이언트의 독립적인 진화가 가능하게된다.

사실 HATEOAS를 지키는것은 웹 테그가 정의되어있는 HTML로 쉽지만

앱은 JSON 을 사용하기에 어렵다.

따라서 독립적인 진화가 가능한 JSON을 검토해봐야한다.

### 정의된 문서 가르쳐주기

    {
        "success": true,
        "code": 0,
        "message": "성공하였습니다.",
        "profile": "<http://server.com/doc/home>",
        "data": 
          {
              "id": "k4keye",
              "jwt": "jwt:s67dsf:jasdk8iop234:odj21389"
          }
    }
    
    <http://server.com/doc/home>
    ""
    API 문서
    
    id 는 너의 아이디야
    jwt 는 앞으로 이걸 쓰렴
    ""

이로인해 이 메시지를 읽는사람은 문서를 보고 id,jwt 의 의미를 알게된다.

### 하이퍼링크로 표현

    {
        "success": true,
        "code": 0,
        "message": "성공하였습니다.",
        "data": 
        {
          "id": "k4keye",
          "jwt": "jwt:s67dsf:jasdk8iop234:odj21389"
        }
        "links": [
            {
                "rel": "home",
                "href": "<http://localhost:8080/home>"
            },
            {
                "rel": "account",
                "href": "<http://localhost:8080/accounts>"
            },
            {
                "rel": "setting",
                "href": "<http://localhost:8080/settings>"
            }
        ]
        
    }

![](d86f7813-f4a1-4193-913c-3a5c8e21add6.png)

PATCH Method 의 등장
-----------------

일반적으로 알고있는 HTTP 의 메소드는 GET,POST,PUT,DELETE 이며

2010년 Ruby on Rails 가 PATCH 필요성을 주장하여 2010 년에 공식 HTTP 메소드로 추가 되었다.

*   기존의 Ruby on Rails 에서 데이터를 편집하는 기준은 기존의 데이터의 일부분만을 수정하여 부분 업데이트 하는 것을 양식을 따르고있고 이를 요구하여 HTTP 메소드로 PATCH 가 추가됨.

PUT 와 PATCH 는 결과적으로는 기존의 있던 자원의 수정을 의미하며

이를 혼용해서 사용하는경우가 있지만

둘은 서로 대체되는 관계가 아닌 다른 정의와 **규약**을 가지고있다.

method

action

PUT

전체 수정

PATCH

일부 수정

### PUT

PUT 메소드는 요청한 URL 에 있는 자원을 대체 하는 메소드이다.

대체 라는 것은 대상을 저장하기도, 변경하기도 한다.

즉 PUT 메소드는 상황에 따라 2가지 방식으로 동작한다.

상황

행위

결과

자원이 존재 할때

해당 자원의 정보 변경

200 혹은 204

자원이 존재하지 않을때

해당 자원 저장(post 와 동일)

201

만약 어떠한 게시물에 대한 좋아요 혹은 싫어요 를 추가할수있는 엔티티가있다고했을때

사용자가 게시물 A 에 대해 좋아요를 누르게된다면

해당 게시물 A 와 사용자로 이미 넣은데이터(싫어요 등)가 있다면 그 데이터를 좋아요로 변경하고(업데이트)

만약 게시물 A와 사용자로 넣은 데이터가 없다면 데이터를 추가하는것으로(추가)

PUT 규약을 지키게된다.

PUT 메소드는 자원이 없으면 저장을 해야하기에 클라이언트가 자원의 상태를 모두 알고있어야만 한다.

만약 클라이언트가 불안전한 Payload (서버는 name 데이터를 원하는데 클라이언트가 nickname 로 전달하게되면) 를 서버에게 전달하게되면

그 일부 entity 의 field 값이 null 로 변경될 위험이 존재한다.

### PATCH

자원에 대한 부분적 수정을 적용하기위한 메소드이다.

위에서의 PUT 는 저장 을 하기위해서 혹은 한번에 변경하기위해서 완전한 자원의 상태를 알고있어야만 했다면

PATCH 는 일부만 수정하기에 필요한 데이터 요청을 payload 로 전달한다.

PATCH 의 명령이 결과적으로 PUT 메소드를 사용하는것보다 큰 크기의 데이터를 전달한다면

이때는 PUT 로 변경해야한다.

### ETC

PUT 와 PATCH 는 HTTP 의 메소드이지만 규약일뿐 이것으로 강제 할수는없다.

하지만 이러한 규약은 모두가 동의한 약속이며 클라이언트와 서버간의 혼란이 발생하지 않도록 정의를 잘 알고 사용해야한다.