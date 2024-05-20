# HTTPCP(Hyper Text Coffee Pot Control Protocol) 


![](510f3229-72dc-49f1-b8ac-9653f8a936ba.png)

RFC 2324과 RFC 7158에 의하면

커피를 만들고 싶어 하는 컴퓨터 엔지니어들을 위한 프로토콜 'HTTPCP'이 정의되어있다

HTTPCP는 HTTP 1.1을 기반으로 동작한다.

> HTTP 1.1 (\[RFC2068\]) permits the transfer of web objects from origin servers to clients. The web is world-wide. HTCPCP is based on HTTP. This is because HTTP is everywhere. It could not be so pervasive without being good. Therefore, HTTP is good. If you want good coffee, HTCPCP needs to be good. To make HTCPCP good, it is good to base HTCPCP on HTTP.

HTTPCP의 프로토콜을 구성하기 위해서는 추가적인 메서드, 헤더, 반환 코드 등 이 필요하며

복잡한 스펙을 가지고 있지만 중요한 몇몇 스펙만 살펴볼 것이다.

### 메서드

BREW or POST

*   media type을 application/coffee-pot-command로 요청한다.
*   해당 메서드는 커피를 끓이게 지원한다.

GET

*   HTCPCP에서 커피는 정보 자원이 아닌 물리적 리소스이기에 커피 자체를 얻는 것이 아닌 커피 정보를 얻는 것이다 따라서 이 정보에는 카페인이 없다

WHEN

*   커피를 따르고 우유를 추가할 때 우유의 양을 조절할 수 있는 기능이다

### 해더

Accept-Additions를 사용하여 양조 방법을 지원하며 설탕의 종류를 설탕, 자일리톨, 스테비아 모두 지원한다.

    addition-type   = ( "*"
                      | milk-type
                      | syrup-type
                      | sweetener-type
                      | spice-type
                      | alcohol-type
                      | sugar-type
                      ) *( ";" parameter )
    sugar-type      = ( "Sugar" | "Xylitol" | "Stevia" )
    

### 418 status code

> Any attempt to brew coffee with a teapot should result in the error code "418 I'm a teapot". The resulting entity body MAY be short and stout.

클라이언트의 요청이 커피 포트가 아닌 차 주전자로 커피를 끓이려고 할 때 발생하는 메시지이다.

springframework의 HttpStatus 객체도 418 응답을 제공한다.

    public enum HttpStatus implements HttpStatusCode {
      I_AM_A_TEAPOT(418, Series.CLIENT_ERROR, "I'm a teapot"),
    }

구글은 HTTPCP 418 메시지를 사용하고 있으며

![](2af1dab1-32fa-4817-a745-e76c2ec1a7ea.png)

Python은 3.9 버전부터 지원한다

![](a3eab00e-7717-47fd-b781-e231306afc41.png)

### 국제적인 커피 지원

커피는 국제적으로 마시는 것이기 때문에 다양한 언어의 커피를 지원해야 한다

    coffee-scheme = (
            "koffie"          ; Afrikaans, Dutch
            | "qæhvæ"        ; Azerbaijani
            | "قهوة"         ; Arabic
            | "akeita"       ; Basque
            | "koffee"       ; Bengali
            | "kahva"        ; Bosnian
            | "kafe"         ; Bulgarian, Czech
            | "caf%C3%E8"    ; Catalan, French, Galician
            | "咖啡"          ; Chinese
            | "kava"         ; Croatian
            | "kC3A1va     ; Czech
            | "kaffe"        ; Danish, Norwegian, Swedish
            | "coffee"       ; English
            | "kafo"         ; Esperanto
            | "kohv"         ; Estonian
            | "kahvi"        ; Finnish
            | "%4Baffee"     ; German
            | "καφέ"         ; Greek
            | "कौफी"         ; Hindi
            | "コーヒー"       ; Japanese
            | "커피"          ; Korean
            | "кофе"         ; Russian
            | "กาแฟ"         ; Thai
            )
    

### 보안 고려 사항

*   인터넷 사용자의 무단 액세스는 “커피 서비스 거부” 공격으로 이어질 수 있다.
*   커피 찌꺼기를 인터넷 배관에 넣으면 막힐 수 있다.

### 마무리

해당 프로토콜은 1998년 4월 1일 만우절에 나온 장난 프로토콜이지만

HTTP status 418의 경우 많은 서버에서 이스터 에그로 이를 구현해 버리게 되어

[RFC 9110에서](https://www.rfc-editor.org/rfc/rfc9110.html#name-418-unused) 영구 결번으로 등록된다.

[모질라](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/418)의 경우 418 응답을 처리하고 싶지 않은 요청에 대해 이를 사용한다고 한다.

참고 및 출처

[https://www.rfc-editor.org/rfc/rfc9110.html#name-418-unused](https://www.rfc-editor.org/rfc/rfc9110.html#name-418-unused)

[https://datatracker.ietf.org/doc/html/rfc2324](https://datatracker.ietf.org/doc/html/rfc2324)

[https://www.google.com/teapot](https://www.google.com/teapot)

[https://developer.mozilla.org/ko/docs/Web/HTTP/Status/418](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/418)

[https://web.archive.org/web/20201007172936/https://docs.python.org/3/whatsnew/3.9.html#http](https://web.archive.org/web/20201007172936/https://docs.python.org/3/whatsnew/3.9.html#http)