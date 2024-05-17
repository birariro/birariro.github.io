# RVA to RAW 쉽게 계산하기 


![](9243f147-8197-4321-8de9-5f791b3320ab.png)

  
파일에서는 Offset로 위치를 표현하고 메모리에서는 address(VA) 로 위치를 표현하는데  
위의사진에서는 파일과 메모리에서의 위치가 차이가난다.  
그래서 메모리에서의 값을 확인한후 그위치를 수정하고자할때는 그위치의  
Offset값을 구하여만 한다.  
  
**RAW = RVA - VirtualAddress(VA) + PointerToRawData**  
  
**RAW****은 파일에서의 주소 이다.**  
**RVA** **는 메모리에서의 주소이다**  
**VirtualAddress(VA)** **은 메모리에서의 섹션시작위치이다.**  
**PointerToRawData** **은 파일에서의 섹션 시작위치이다**

![](c9a03b6b-5c32-4512-88b5-4d3caa67bb91.png)

위에서 RVA는 VA - ImageBase 로 구할수있는데 여기서 ImageBase 는 위에 사진에서의 메모리 시작 address를 보면  
0부터 시작안하고 010000000 로시작하는데 이때 01000000이 ImageBase 이다.  
즉 시작 점에서의 RVA는 0인 것이다.  
  
이제 문제로 RVA = 300 일때 offset(RAW) 를 구해보자.  
  
RVA 3000 이면 위의 메모리 에서 섹션을 보면  
  

![](700990cd-1eb1-42ce-8886-3d7a11b58f59.png)

.test섹션 에 포함되어있다는걸 쉽게할수있다  
\=01001000 인데 왜 포함되냐 라고생각한다면 여기서 ImageBase를 빼서 생각해야한다.  
  
VirtualAddress(VA) 은 메모리에서의 섹션 시작 위치이니  
.test의 섹션시작위치를 보면된다 즉 1000이다.  
  
PointerToRawData 은 파일에서의 섹션 시작위치이니  
.test의 섹션시작위치를보면

![](189333a5-2d9e-4322-86ee-9ba7f8a71c5a.png)

400이라는걸 알수있다. 파일에서는 offset 은 ImageBase를 포함하고있지않으니 그대로 보면된다.  
  
이제 식을 대입해보면  
**RAW = RVA - VirtualAddress(VA) + PointerToRawData**  
  
**\-----> 3000(RVA) - 1000(VA) + 400(PointerToRawData) = 2400(RAW)**  
  
가 된다.  
  
  
이미지 출처: 리버스코어