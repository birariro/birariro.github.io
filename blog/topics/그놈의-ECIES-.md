# 그놈의 ECIES 


![](55a85ff6-b862-4433-99bf-a4cb7120623b.png)

**TL;DR**
=========

1.  엘리스는 자신의 개인키와 밥의 공개키로 대칭키를 만든다.
2.  대칭키로 암호문을 생성하고 암호문의 무결성을 검사할수있는 값을 만들어 암호문과 함께 보낸다.
3.  밥은 자신의 공개키와 엘리스의 개인키로 대칭키를 만든다.
4.  암호문의 무결성을 검사한다.
5.  대칭키로 엘리스의 암호문을 해독한다.

**Elliptic Curve Integrated Encryption Scheme**
===============================================

ECIES 는 Hybrid Encryption Scheme 중에 하나로  
Elliptic Curve 즉 타원곡선을 사용하는 암호를 의미하며  
Encryption Scheme는 데이터를 암호화하여 전송하기위한 모든 과정을 나타내는 의미를 가진다.  
즉 ECIES는 타원곡선을 활용하여 임의의 대칭키를 생성하고 상대의 공개키로 암호화해 전송후 이를 복호화해 키 교환 단계를 거치고 서로 합의한 대칭키 암호화 방식을 사용하는것이다.  
ECIES 는 타원곡선 암호화 알고리즘은 고정되어있지만 나머지 암호화 알고리즘이 고정되어있지않다 즉 원하는 암호 알고리즘을 사용할수있다. 공개키 생성을 secp256k1, P-521 을 사용해도되고  
대칭키 암호화 알고리즘으로 AES-CTR, ARD-GCM 을 사용해도된다.  

**용어**
======

Key Agreement (KA)
------------------

키 합의 라고부른다.  
서로가 개인키를 숨긴상태에서 타인이 모르는 바이트 열인 shared secret를 만드는 작업이다.  
이 작업이 위의 사이클에서는 암호키와 MAC키를 만드는 작업으로 보인다.

Key Derivation function (KDF)
-----------------------------

다른키에서 새로운 키 여러벌을 생성하는 함수를 말한다.  
키 유도 함수 라고부른다.  
KDF의 조건은 아래와 같다.

1.  새로운 키는 암호화에 사용가능한 키여야 한다.
2.  새로운키에서 원래의 키를 알수없다.
3.  키는 한개만 나올수있는게아니다 따라서 입력과 함께 임의의 문자열을 넣을수있다.

Message Authentication Code (MAC)
---------------------------------

메시지 인증 코드라 부르며 메시지의 인증에 쓰이는 작은 크기의 값이다.  
키를 사용하여 임의의 길이의 메시지를 인증할수있는 MAC Tag를 출력한다.  
MAC Tag 는 같은 키를 사용하여 무결성을 보호할수있다.  
일반적으로 암호문에 인증 코드를 붙히는게 일반적이다.  
데이터가 무한히 길어질수있기에 보통 해시를 먼저하고 그 결과를 MAC 함수에 돌린다  

**사이클**
=======

1.  **시작**
    1.  엘리스 는 공개키와 개인키를 생성한다.
    2.  밥 은 공개키와 개인키를 생성한다.
    3.  엘리시는 밥에게 메시지 M을 조심스럽게 전달하려한다.
2.  **암호화**
    1.  엘리스는 자신의 개인키와 밥의 공개키를 사용하여 Key Agreement 를 수행한다.
        1.  Key Agreement 으로써 Key Derivation function 를 진행한다.
    2.  엘리스 자신의 개인키와 밥의 공개키를 Key Derivation function 의 입력값으로 사용하여 충분한 길이의 바이트 열을 얻고 이것을 암호키와 MAC키로 분활한다.
    3.  (2-b) 에서 얻은 암호키로 보낼 메시지 M 을 대칭키 암호화 알고리즘으로 암호화하여 암호문 C 를 얻는다.
    4.  (2-c) 에서 얻은 암호문 C 를 MAC 입력값으로 사용하여 MAC Tag를 생성한다.
        1.  MAC 에는 암호문 C와 MAC키가 들어가는것으로 예상된다.
    5.  결과적으로 MAC Tag, 암호문 을 밥에게 전달한다.
3.  **검증**
    1.  밥은 자신의 개인키와 엘리스의 공개키로 Key Agreement 를 수행한다.
    2.  암호키와 MAC키 를 얻은 밥은 MAC키와 암호문으로 MAC Tag를 계산하고 엘리스가 보내준 MAC Tag와 같은지 비교하는것으로 엘리스가 보낸것이라는것을 확신한다.
    3.  암호문 을 암호키로 복호화 하여 메시지를 얻는다.

참고 자료
-----

[https://cryptobook.nakov.com/asymmetric-key-ciphers/ecies-public-key-encryption](https://cryptobook.nakov.com/asymmetric-key-ciphers/ecies-public-key-encryption)  
[https://gist.github.com/aJchemist/f2d08f328f0458be8ee8](https://gist.github.com/aJchemist/f2d08f328f0458be8ee8)  
[https://ko.wikipedia.org/wiki/메시지\_인증\_코드](https://ko.wikipedia.org/wiki/메시지_인증_코드)