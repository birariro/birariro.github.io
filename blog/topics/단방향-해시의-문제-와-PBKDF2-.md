# 단방향 해시의 문제 와 PBKDF2 


**TL;DR**
=========

1.  단방향 해시는 쉽게 해킹당할수있다 따라서 솔팅 이나 키스트레칭으로 이를 해결한다.
2.  PBKDF2는 패스워드를 솔팅이라는 값과 함께 여러번 해시를 하는 작업이다.

**단방향 해시**
==========

**문제점**
-------

대부분 패스워드 관리와 같은 중요한 정보는 해시를 통해 저장을하게되는데

사실 해시는 패스워드 저장에 설계된것이 아닌 매우 빠른 처리속도로 데이터를 검색하기위해 설계되어 이로인해 해킹공격또한 가능해지게된다.(레인보우 테이블 획득 속도 증가)

**해결 방안**
---------

### 1\. 솔팅(salting)

솔트(salt)는 다이제스트(해시된 값) 을 생성할때 추가하는 임의의 문자열을 말한다.

만약 패스워드 q1w2e3r4 를 해시할때 솔팅 작업을 한다고하면

(raonsalt(솔트) + q1w2e3r4 (패스워드)) → 해시 → 결과

의 작업을 수행하다고 보면된다.

💡 쉽게 생각해서 솔트는 해시할때 사용하는 암호키(이 용어를쓰면 안되겠지만) 라고 볼수있다.

### 2\. 키 스트레칭(Key Stretching)

입력한 패스워드의 다이제스트를 생성하고 생성된 다이제스트를 사용하여 다시 다이제스트를 생성하고 이를 반복... 하는 방법을 말한다.

![](73984d2d-42fa-4509-bec8-4c54b2ebeb03.png)

결과 패스워드를 동일한 횟수만큼 해시해야만 일치여부를 확인할수있게된다.

최근 일반적인 장비로 1초에 56억개의 다이제스트를 비교할수있지만

잘 설계된 패스워드 시스템에는 다이제스트 생성시간이 0.2초 이상 걸리게 설정하여

일부러 소요시간을 증가시키는 방식을 사용해서 1초에 5번만 비교할수있게 한다.

**Adaptive Key Derivation Functions**
=====================================

AKDF 는 다이제스트를 생성할때 솔팅과 키 스트레칭을 반복하며 솔드와 패스워드 외에도 입력값을 추가하여 공격자가 쉽게 다이제시트를 유추할수없도록 하는 방법이다.

그중 가장 많이 사용하는 방식이 PBKDF2 이다.

Password-Based Key Derivation Function Version 2
------------------------------------------------

PBKDF2는 솔트를 적용후 해시함수의 반복횟수를 임의로 선택할수있다.

함수의 기본 파라미터는 아래와 같다.

    PBKDF2(난수, 패스워드, 솔트, 반복횟수, 원하는 다이제스트 길이)
    

![](208d5d75-9650-4f4f-9182-839bdf52e6f1.png)

**의문점**
=======

해시를 정한다면 왜 다이제스트 길이를 정할까?
-------------------------

해시를 sha256을 선택하면 어차피 키는 256비트가 나올텐데 왜 키길이를 정하는지

[https://stackoverflow.com/questions/58508998/pbkdf2withhmacsha256-impact-of-key-length-to-the-output-length](https://stackoverflow.com/questions/58508998/pbkdf2withhmacsha256-impact-of-key-length-to-the-output-length)

위의 사람도 위와같은 의문을 가졌다

다이제스트 길이를 512로 지정하고 해시 알고리즘을 sha 256으로 하면

해시 길이로인해 256의 키가 나올줄알았는데 512길이가 나온다.

PBKDF2는 기본 해시보다 큰 출력을 내기때문에 다이제스트 길이를 맞춰주게된다.