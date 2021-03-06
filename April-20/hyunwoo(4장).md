<a href="https://www.notion.so/HTTP-4-36cfdf43522649f69e2d4b57c0cd4d18">정리 원본</a>
  
  # [HTTP 완벽 가이드] 4장.  커넥션 관리

- 4장에서 다루게 될 주제를 바탕으로 정리한 글이다.
    - HTTP는 어떻게 커넥션을 사용하는가
    - TCP 커넥션의 지연, 병목, 막힘
    - 병렬 커넥션, Keep-alive 커넥션, 커넥션 파이프라인을 활용한 HTTP의 최적화
    - 커넥션 관리를 위해 따라야 할 규칙들
    

# TCP 커넥션과 지연 상황🔗

우선 우리는 “TCP”과 “커넥션”이란 무엇인지 간단하게 알고 갈 필요가 있다.

## TCP(Transmission Control Protocol)란? 🤷‍♂️

: HTTP 네트워크 프로토콜 스텍에서 ‘전송 계층', 패킷 교환 네트워크 프로토콜의 집합 ⇒ 교환이라는 것은 즉, 양방향(클라이언트 - 웹 서버)으로 이루어진다라는 것.

** 전송계층 : End Point간 신뢰성있는 데이터 전송을 담당하는 계층

<aside>
❓ 패킷(Packet)이란? 
정보를 전송할 때 특정 형태를 맞춰서 보내는 데이터 조각, 컴퓨터 간 데이터를 주고 받을 때 네트워크를 통해서 전송되는 데이터 조각이다. 계층마다 데이터의 형식이 다르다.

</aside>

![transport layer](https://user-images.githubusercontent.com/96808980/164067554-8c7c1380-72f3-4d63-9ad9-fdbabb84830e.png)

### TCP의 역할

TCP는 “**신뢰성 있는 데이터 통신**을 가능하게 해주는 프로토콜”이다.

- 오류 없는 데이터 전송
- 순서에 맞는 전달 - 데이터는 언제나 보낸 순서대로 도착한다.
- 조각나지 않는 데이터 스트림 - 언제든 어떤 크기로든 보낼 수 있다.

## 커넥션(conection)이란? 🤷‍♂️

송신자와 수신자의 논리적 연결을 생성하는 것을 의미한다.

쉽게 생각하면 클라이언트와 서버간의 전용 통로를 만들어 서로 대화하는 것이라고 생각하면 쉽다.

 `신뢰성 있는 데이터 통신`이라고 언급한 이유는 커넥션이 맺어지면 클라이언트와 서버 컴퓨터 간에 주고받는 메시지들은 손실이나 손상되거나 순서가 바뀌지 않고 안전하게 전달된다.

## 그럼 TCP 커넥션의 특징은?

### 1. 커넥션을 연결한다.

**TCP의 가장 중요한 특징 “커넥션”을 연결한다는 것이다.**

TCP 커넥션은 인터넷을 안정적으로 연결해주고, TCP 커넥션의 한 쪽에 있는 byte들은 반대쪽으로 순서대로 정확히 전달된다.

클라이언트는 웹 서버에 TCP 커넥션을 이용해 데이터를 전송할 경우

`GET / index.html HT ...` → `...TH lmth.xedni / TEG` 로 반대로 정렬하여 전송하기 때문에 순서대로 데이터(문자)를 받을 수 있는 장점이 있다.

### 만약 순차적으로 데이터를 보내지 않는다면?

데이터의 손실이 발생하여 원하는 데이터의 전송을 받을 수 없다.

---

### 웹브라우저가 TCP 커넥션을 통해 웹 서버에 요청을 보내는 과정

| 요청 과정 | 예시 |
| --- | --- |
| 브라우저가 호스트 명을 추출하여 에서 IP를 읽어옴 | http://www.joes-hardware.com:80/power-tool에서 ‘호스트’를 IP인 202.43.78.3으로 변환  |
| IP와 포트 번호로 커넥션을 생성(만약 포트번호가 생략되어있다면 기본 포트번호 80번을 사용) | 202.43.78.3:80으로 커넥션을 연결 |
| 브라우저가 서버로 HTTP GET 요청 메세지를 전송 | 클라이언트가 웹 서버에 요청 |
| 서버에서 온 HTTP 응답 메세제를 읽음 | 웹 서버에서 클라이언트에게 응답 |
| 브라우저가 페이지를 띄워줌 | 브라우저가 페이지를 띄워줌 |
| 브라우저가 TCP 커넥션을 닫음 | 클라이언트가 커넥션을 끊음 |

---

### 2. TCP 커넥션을 통해 웹 서버에 데이터를 보낼 경우 “세그먼트”를 나눈다.

![segment](https://user-images.githubusercontent.com/96808980/164067618-12499e9f-13e0-45f1-823f-6d299ddca1e1.png)
  
웹 브라우저가 TCP 커넥션을 통해 웹 서버에 데이터를 전송할 때 위의 이미지처럼 

1. 전송할 데이터를 나눠서 
2. IP 패킷 헤더(보통 20바이트), TCP 세그먼트 헤더(보통 20바이트), TCP 데이터 조각(0이나 데이터 크기에 따라 다름)을 “IP 패킷”이라는 봉투에 담아
3. 웹 서버에 전송한다.

IP 패킷을 구성하는 각각의 데이터 들을 ‘세그먼트’라고 한다.

*** **TCP헤더의 구조**

![TCPheader](https://user-images.githubusercontent.com/96808980/164067672-bcdc1e47-ef44-4f3f-a6e7-9df13d9452f3.png)
  
[TCP의 헤더에는 어떤 정보들이 담겨있는걸까?](https://evan-moon.github.io/2019/11/10/header-of-tcp/)

TCP헤더는 발신지 IP주소, 목적지 IP주소, 시퀀스 번호(Sequence number), 승인 번호(Acknowledgement number), 플래그(Flag, NS~FIN) 등으로 구성이 된다.

시퀀스 번호는 데이터의 전송 순서의 의미이며, 역할은 전송하는 데이터마다 번호를 붙여 언제든 순서대로 정렬할 수 있게 해준다.

승인 번호는 수신자의 입장에서 예상하는 시퀀스 번호이다. 데이터를 전송하기 위해 커넥션을 열고 닫을 때는 `시퀀스 번호+1`이다.

### 3. 컴퓨터는 항상 TCP 커넥션을 여러 개를 유지한다.

포트 번호를 통해 여러 개의 커넥션을 유지한다.

여러 개의 커넥션은 `네 가지 값`으로 식별한다.

```markdown
<발신지 IP주소, 발신지 포트, 수신자 IP주소, 수신자 포트>
```

중요한 점은 네 가지 주소 구성요소의 값이 일부는 같을 수 있지만 모두 같을 수 없다는 것이다.

커넥션은 “**중복될 수 없는 유일한 통로를 만드는 것**”이라 생각할 수 있다.

### 4. TCP 소켓 프로그래밍

<aside>
❓ 소켓이란?
네트워크로 데이터를 전송하거나 네트워크로 부터 데이터를 받기 위한 실제적인 창구이다. TCP 종단(EndPoint)에 데이터 구조를 생성하고 데이터 스트림을 읽고 쓸 수 있다.

</aside>

TCP 소켓 프로그래밍은 쉽게 TCP가 데이터 교환을 할 때 **소켓을 사용한다는 의미**이다.

![Frame 7](https://user-images.githubusercontent.com/96808980/164067740-90363084-03e5-47ba-bce6-17a1114b35f6.png)

위처럼 소켓은 TCP의 종단에 위치하는데 데이터가 나가고 데이터가 들어오는 첫번 째 관문이다.

반드시 소켓이 열려있어야 데이터의 전송과 수집이 가능하다.

[소켓 프로그래밍. (Socket Programming)](https://recipes4dev.tistory.com/153)

---

## TCP의 성능에 대한 고려(지연)

![transport layer](https://user-images.githubusercontent.com/96808980/164068012-d7566d9a-e909-448b-9658-afb9f29e3550.png)

HTTP는 TCP 바로 위의 계층이다. 그래서 HTTP의 트랜잭션의 성능은 그 아래 계층인 `TCP 성능`에 영향을 받는다.

### 트랜잭션 지연

![[https://feel5ny.github.io/2019/08/26/HTTP_004_01/](https://feel5ny.github.io/2019/08/26/HTTP_004_01/)](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/72fa07ed-7b63-41d9-8a44-fea8330e55fe/Untitled.png)

[https://feel5ny.github.io/2019/08/26/HTTP_004_01/](https://feel5ny.github.io/2019/08/26/HTTP_004_01/)

대체적으로 트랜잭션이 처리되는 시간은 상당이 짧다.

하지만 클라이언트나 서버가 너무 많은 데이터를 내려 받거나 복잡하고 동적인 자원들을 실행하지 않는 한 대부분의 HTTP 지연은 TCP 네트워크 지연이다.

자세하게 말하면,

1. IP주소와 포트 번호를 알아내야 하는데 호스트에 방문한 적이 최근에 없다면 DNS 이름 분석을 해서 URI에 있는 호스트명을 주소로 변환하는데 수십 초의 시간이 소요
2. 새로운 TCP 커넥션에서 항상 발생하는 커넥션 설정 시간은 보통 1~2초지만 수백 개의 HTTP 트랜잭션이 많들어져 소요시간은 크게 증가함
3. 요청 메세지가 인터넷을 통해 전달되고 서버에 의해 처리되는데 시간이 걸림
4. 웹 서버가 HTTP 응답을 보내는 것도 역시 시간이 소요됨
5. 부가적으로 하드웨어의 성능, 네트워크와 서버의 전송 속도, 요청과 응답 메시지의 크기, 클라이언트와 서버 간의 거리에 따라 크게 달라짐

[Submarine Cable Map](https://www.submarinecablemap.com/)

클라이언트와 전세계의 서버간의 거리를 볼 수 있음 

### 성능 관련 중요 요소(일반적인 TCP 지연의 경우)

- **TCP 커넥션의 핸드셰이크 설정**

![handshaking](https://user-images.githubusercontent.com/96808980/164068221-d4f0592f-fbc7-4035-9ada-ea7d60f00568.png)

** 핸드셰이크 = **패킷을 주고 받기**

**TCP 커넥션 내에서 일어나는 클라이언트와 서버와의 IP 패킷의 교환하는 과정**이 악수(handshaking)과 유사해서 붙여진 이름

TCP헤더의 구조에서 구성되어 있는 플래그가 핸드셰이크 상황에서 사용된다 

플래그는 TCP 연결 제어 및 데이터 관리를 담당한다.

“SYN”이라는 것은 “커넥션 생성 요청”을 의미하고

“ACK”는 클라이언트에서 온 요청을 “승인 확인”한다는 응답 메세지라고 생각하면 된다.

데이터를 전송할때마다 TCP 커넥션을 연다고 생각을 해보면, 클라이언트와 웹 서버스는 요청/응답의 메세지를 계속 보내면서 IP 패킷을 교환하고 커넥션을 열게 될 것이다.

그리고 작은 데이터임에도 불구하고 요청/응답이 많아지면 많아질 수록 HTTP 성능이 크게 떨어질 수 있다.

**데이터 크기는 작을 수록, 데이터의 전송 횟수가 많을 수록 지연이 발생한게 체감이 된다고 함.**

[핸드셰이킹 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%ED%95%B8%EB%93%9C%EC%85%B0%EC%9D%B4%ED%82%B9)

- 확인응답(ACK) 지연

인터넷 자체에서는 패킷 전송의 성공을 완벽히 보장하지 않는다. (라우터가 과부하시 패킷을 마음대로 파기)

하지만 TCP는 신뢰성 있는 전송을 보장하기 때문에 성공적인 데이터 전송을 위한 **자체적인 확인 체계**를 가진다.

TCP는 ‘세그먼트’라는 것을 ‘IP 패킷’이라는 봉투에 담아 전송을 한다고 했다.

그 세그먼트에는 순서대로 데이터를 전송했을 때 성공적이라면 서버에서 확인 응답(ACK)을 받는다.

만약 송신자, 즉 클라이언트가 특정 시간(보통 0.1~0.2초) 안에 서버로 부터 응답 메세지를 받지 못하면 패킷이 파기 되었거나 오류가 있는 것으로 간주하고 데이터를 다시 전송한다.

![response](https://user-images.githubusercontent.com/96808980/164068293-3fac490c-b036-4a67-b907-243f22928c96.png)

응답 메세지가 지연이 되는 이유는 확인응답 지연알고리즘 때문이다.

확인응답은 크기가 매우 작기 때문에 TCP는 같은 방향으로 송출되는 데이터 패킷에 확인응답을 편승시키려고 한다.

그래서 확인응답을 특정 시간동안 “버퍼”에 저장해두고 편승시킬 송출 데이터 패킷을 찾는다.

만약 일정 시간 안에 송출 데이터 패킷을 찾지 못하면 확인응답은 별도의 패킷을 만들어 전송한다.

하지만 요청과 응답 두 가지의 형식으로만 이루어지는 HTTP 동작 방식은 확인 응답이 송출 데이터 패킷을 편승할 기회를 감소시킨다.

그래서 송출 데이터에 편승시키려는 “확인응답 지연 알고리즘”으로 인해 지연이 발생하는 것이다.

- TCP 느린 시작

TCP 커넥션은 시간이 지나면서 자체적인 “튜닝"이 된다. 

인터넷이 갑작스런 부하와 혼잡이 일어날 수가 있는데 그것을 방지하기 위해

처음에는 최대 속도를 제한하다가 데이터가 성공적으로 전송이 되면 될 수록 속도 제한을 푼다.

이렇게 조율하는 것은 “TCP 느린 시작”이라고 한다.

또한 한 번에 전송할 수 있는 패킷의 수를 제한한다. 패킷의 수 또한 데이터 전송에 결과에 따라 증가한다.

이런 “느린 시작"으로 인해서 튜닝이 되기 전 새로운 커넥션은 느릴 수 밖에 없다.

- 존 네이글의 네이글(Nagle) 알고리즘

각 TCP 세그먼트는 40byte 상당의 플래그와 헤더를 포함하여 전송한다.

근데 만약 작은 크기의 데이터를 하나씩 패킷에 담아 많이 전송한다면 네트워크 성능은 크게 떨어질 것이다.

그래서 네이글 알고리즘은 네트워크의 효율을 위해 패킷 전송 전에 많은 양의 TCP 데이터를 한 개의 덩어리로 합치는데

만약 세그먼트가 최대 크기(약 1,500byte)를 채우지 못하면 전송을 하지 않고 버퍼에 쌓아둔다.

그 후에 서버로 부터 확인응답이 도착하면 쌓아둔 모든 세그먼트를 패킷으로 감싸서 전송한다.

이것은 성능에 큰 문제를 야기하는데,

1. 크기가 작은 작은 HTTP 메세지는 패킷을 채우지 못하기 때문에 언제 생길지도 모르는 추가 데이터를 채울 때까지 무한 대기 상태
2. 만약 ‘확인응답 지연 알고리즘’이 함께 활성화 되었다면 네이글 알고리즘의 지연시간과 ‘확인응답 지연 알고리즘’으로 인한 지연시간이 더 추가된다.

이런 문제를 해결하기 위해 “TCP_NODELAY” 파라미터 값을 설정해서 네이글 알고리즘을 비활성화 시킬 수 있다. 

![Nagle](https://user-images.githubusercontent.com/96808980/164068388-83c2ca3d-e060-447d-98f1-0d05d99cc4cb.png)

네이글 알고리즘 활성화와 비활성화의 차이

- TIME_WAIT

TCP 커넥션의 종단에서 커넥션을 끊으면 종단에서 IP주소, 포트 번호를 기록해 놓는데 이는 같은 주소와 포트 번호를 사용하는 새로운 커넥션이 일정시간(보통 2분 정도) 동안 생성되는 것을 방지하기 위함이다.

커넥션을 끊으면 TIME_WAIT이라는 포트에 들어가게 되는데 “연결을 끊은 IP주소와 포트 번호를 가지고 있는 커넥션”이 많다면 대기 상태가 늘어나 극심하게 느려지는 상황이 나올 것이다.

하지만 현대의 빠른 라우터의 성능과 여러 개의 가상 IP주소를 생성할 수 있어 문제가 발생하는 것은 극히 드물어서 알고만 가는 걸로 하자.

# 병렬(Paralled) 커넥션

여러 개의 TCP 커넥션을 통한 동시 HTTP 요청하는 데이터 교환 방식이다.

![parallel](https://user-images.githubusercontent.com/96808980/164068468-048eaab8-a095-4d28-bd8e-bc3fc7fddf35.png)

즉 클라이언트가 여러 개의 서버에 여러 개의 커넥션을 맺어서 여러 개의 HTTP 트랜잭션을 병렬로 처리할 수 있다.

## 병렬커넥션의 특징

![병렬](https://user-images.githubusercontent.com/96808980/164068542-d0ab295e-d134-4cba-b9cd-46ce37317163.png)

- 페이지를 더 빠르게 내려받는다.

: 이것은 그림만 보아도 알 수 있다. 당연히 여러 개의 커넥션을 통해 HTTP 요청과 응답 메세지가 전송될 것이고 처리될 것이기 때문에 빠를 수 밖에 없다.

- 하지만 병렬 커넥션이 항상 더 빠른 것은 아니다.

:  왜냐하면 클라이언트의 대역폭(네트워크 통신 속도)가 좁으면 하나의 커넥션으로 받는 것이 더 빠를 수 있다.

제한된 대역폭 내에서 각 객체를 전송받는 것은 어처피 느리기 때문에 성능상의 장점이 사라진다.

- 병렬 커넥션은 더 빠르게 느껴질 수 있다.

: 우리가 브라우저를 실행했을 때 로딩 중인 화면을 본 적이 있을 것이다.

브라우저가 병렬 커넥션을 이용하여 순서대로 하나 하나씩 출력하는 것을 보는 것은

사용자에게 더 빠르게 내려받고 있다는 느낌을 준다.

그래서 처음 저해상도의 이미지를 점차 해상도를 높이는 이미지 로딩 방식으로 효과가 극대화될 수 있다.

## 병렬 커넥션 단점

- 각 트랜젝션마다 새로운 커넥션을 맺고 끊기 때문에 시간과 대역폭이 소요한다.
- 각각의 새로운 커넥션은 TCP 느린 시작 때문에 성능이 낮아진다.
- 실제로 연결할 수 있는 병렬 커넥션의 수에는 제한이 있다. (대부분 4개, 최신 브라우저는 5~6개)

# 지속 커넥션

![지속커넥션](https://user-images.githubusercontent.com/96808980/164068603-7fd82eae-cd08-43ac-834a-b0ff7e6ddf37.png)

커넥션을 맺고 끊는 데서 발생하는 지연을 제거하기 위해 커넥션을 재활용하는 방식

## 지속 커넥션의 장점

- 커넥션을 맺기 위한 사전 작업과 지연을 줄여주고 “튜닝"된 커넥션을 유지(느린 시작 참고)한다/
- 병렬 커넥션보다 적은 커넥션을 사용한다.

## 지속 커넥션의 단점

- 지속 커넥션은 데이터 전송을 완료해도 연결 상태를 유지하기 때문에 많은 커넥션이 쌓일 수 있다.

위의 단점을 해결하기 위해 `keep-Alive`가 있다.

![keepAlive](https://user-images.githubusercontent.com/96808980/164068717-9df7b661-f5fa-4884-8ac1-c21c65a27cf3.png)

일일이 커넥션을 연결하는 것보다 트랜잭션을 처리하는 것이 빨라보인다.

그래서 HTTP1.0+는 “Keep-alive” 커넥션을 사용하고 HTTP1.1에는 “지속 커넥션"을 사용한다.

Keep-alive의 버전이 HTTP1.1에서 설계가 수정되어 사실상 이 둘은 명칭이 다르지만 역할은 똑같다.

# Keep-Alive

## Keep-Alive 의 동작

![conection-keepalive](https://user-images.githubusercontent.com/96808980/164068783-044ebcc8-b4b2-4374-a059-30b5b948e588.png)

keep-alive는 사용하지 않기로 결정되어 HTTP1.1 명세에서 빠졌지만 아직 브라우저와 keep-alive 핸드셰이크가 널리 사용되고 있어 알아둬야 한다.

일단 keep-alive는 HTTP1.0+ 버전이라는 것을 알아두자.

클라이언트는 커넥션을 유지하기 위해 요청에 “Connection:Keep-Alive” 헤더를 포함시킨다.

그 요청을 받은 서버는 그 다음 요청도 이 커넥션을 통해 받길 원한다면 응답 메세지에 같은 헤더를 포함시켜 응답한다.

만약에 “Connection:Keep-Alive” 헤더가 없다면, 이번 데이터 전송이 종료되면 커넥션을 끊는다고 간주한다.

그리고 커넥션이 끊어지기 전에 엔터티의 본문의 길이(Content-length)를 알 수 있어야 커넥션을 유지할 수 있다.

본문의 길이와 함께 멀티 파트 미디어 형식(MIME의 한 종류), 청크 전송 인코딩으로 인코드가 되어야한다.

<aside>
❓ 청크 전송 인코딩
: 메세지를 일정 크기의 단위로 쪼개서 순차적으로 서버로 전송한다.
이는 본문이 동적으로 생성되기 때문에 메세지를 보내기전에 본문의 길이를 알 필요가 없다.

</aside>

## Keep-Alive 옵션

Keep-Alive는 옵션을 설정할 수 있다.

하지만 옵션은 옵션일 뿐 이대로 동작된다는 보장은 없다.

`timeout 파라미터` : 커넥션이 얼마간 유지될 것인지 응답 헤더를 통해 전송한다.

`max 파라미터` : 커넥션이 몇 개의 트랜잭션을 처리할 때까지 유지할 것인지 응답 헤더를 통해 전송한다.

```markdown
Connection: Keep-Alive
Keep-Alive : max=5, timeout=120

5개의 트랜잭션을 처리할 때까지 유지할 것이고 120초간 유지한다.
```

## 멍청한 프락시(dumb)

멍청한 프락시는 Connection 헤더를 인식하지 못한다.

하지만 헤더라는 것을 인지 못할 뿐 전달은 한다.

아래 그림을 보면 이해가 빠를 듯 하다.

![dumb](https://user-images.githubusercontent.com/96808980/164068864-60e24a1c-f998-4879-8284-a841d1d2ca55.png)

멍청한 프록시는 마지막 이해하지 못한 Connection 헤더를 보냈을 때 클라이언트는 프록시를 통해 요청 메세지를 보내서 응답을 받았기 때문에 커넥션을 유지하는 것에 동의했다고 생각한다.

하지만 프록시는 Connection 헤더를 인식하지 못하기 때문에 자신이 모든 데이터 전송이 끝난 줄 알고 커넥션이 끊기는 것을 기다린다.

여기서 서버는 연결 유지를 하는 것으로 알기 때문에 커넥션을 끊지 않는다.

커넥션을 유지하는 것으로 아는 클라이언트는 데이터를 전송하지만 커넥션이 끊어진 커넥션에 걸려서 요청은 무시되며 브라우저는 응답이 없이 로드 중이라는 표시만 나온다.

이후 브라우저는 자신이나 서버가 타임아웃이 나서 커넥션 끊기기를 기다린다.

영리한 프록시가 있음에도 멍청한 프록시와의 커넥션을 거치면 이 문제가 발생한다.

# 파이프라인 커넥션

HTTP1.1은 지속 커넥션을 통해 요청 파이프라이닝을 할 수 있다.

여러 개의 요청은 응답이 도착하기 전까지 큐에 쌓인다.

첫 번째 요청이 네트워크를 통해 서버로 가게 되면 이어서 두 번째, 세 번째 요청이 전달된다.

이것으로 인해 요청의 대기 시간과 응답 소요시간을 줄일 수 있어 성능을 높여준다.
  
![파이프라인](https://user-images.githubusercontent.com/96808980/164068947-08645a34-c66f-4469-b0e8-b62ce27d11c3.png)

# 커넥션 끊기

- 마음대로 커넥션 끊기

: 클라이언트, 서버(클라이언트나 네트워크 실패일 때 제외), 프록시는 언제든 TCP커넥션을 끊을 수 있지만,

 보통 커넥션은 메세지를 다 보낸 다음 끊지만, 에러가 있는 상황에서는 헤더 중간이나 다른 엉뚱한 곳이서 끊길 수도 있다.

- Content-length와 Truncation(끊기)

: HTTP 응답은 본문의 정확한 크기 값을 가지는 Content-Length 헤더를 가지고 있어야한다. 

일부 오래된 HTTP 서버는 자신이 커넥션을 끊으면 데이터 전송이 끝났다고 간주하기 때문에 Content-Length 헤더를 생략하거나 잘못된 길이 정보가 전달될 수 있다.

만약 클라이언트나 프락시가 커넥션이 끊어졌다고 응답을 받으면 실제 전달받은 엔더티 길이와 Content-Length의 값이 일치하지 않거나 생략되어 있다면서버에게 정확한 길이를 물어봐야한다.

(어디서 부터 연결이 끊겼는지에 대한 정보를 찾는건가?)

- 커넥션 끊기의 허용, 재시도, 멱등성

<aside>
❓ 멱등성이란 여러 번 반복했을 때 결과가 항상 같은 성질을 말한다.

</aside>

: 에러가 없어도 커넥션은 언제든 끊을 수 있다. 다만 예상치 못하게 커넥션이 끊어졌을 때 HTTP 애플리케이션에 대한 대응의 준비를 반드시 해야한다.

클라이언트가 트랜잭션을 수행 중 커넥션이 끊기면, 클라이언트는 다시 그 트랜잭션을 재시도해서 문제가 없다면 커넥션을 다시 맺고 한번 더 전송을 해야한다.

클라이언트가 트랜잭션을 수행하기 때문에 “요청”이다.

요청 메서드 중 “GET”, “HEAD”, “PUT”, “DELETE”, “TRACE”, “POST”가 있는데 “POST”를 제외한 나머지가 멱등성을 가진다.

“POST”는 비멱등성을 가진다.

내가 이해한 것으로는 “POST를 제외한 나머지 요청 메서드”는 버튼 클릭 등 여러번 행동을 해도 똑같은 결과가 나오는 것을 멱등성이라고 하고

폼 작성 중 에러 발생 등으로 커넥션이 끊겼을 때 개인정보 등이 기록된 결과를 다시 보여주는 것은 보안 상 좋지 않기 때문에 재시도를 하면 안되는 비멱등성이라고 이해했다. 

- 우아한 커넥션 끊기

![connection](https://user-images.githubusercontent.com/96808980/164069015-045c4d67-a46f-4761-9975-67f1cc863f28.png)

(a)는 `close()`를 호출하여 입출력 채널의 커넥션을 모두 끊었고 (b,c)는 `shutdown()`을 호출하여 입력/출력 채널 중 출력 채널을 선택하여 커넥션을 끊었다.

## TCP 끊기와 리셋 에러

전체 끊기를 할 수 있지만 애플리케이션이 각기 다른 HTTP 클라이언트/서버/프록시와 통신할 때, 파이프라인 지속 커넥션을 사용할 때

기기들의 예상치 못한 쓰기 에러를 발생하는 것을 예방하기 위해 ‘절반 끊기'를 사용해야한다.

보통 커넥션의 출력 채널을 끊는 것이 안전하다. 반대편의 기기가 데이터 전송이 끝남과 동시에 당신의 커넥션이 끊겼다는 것을 인지할 수 있기 때문이다.

만약 클라이언트가 커넥션을 끊은 입력 채널에 데이터를 전송하면 `connection reset by peer`라는 에러 메세지를 클라이언트에게 보낸다.

대부분 운영체제는 이것을 심각한 에러로 취급해 아직 읽히지 않은, 버퍼에 저장된 데이터를 모두 삭제한다.

### 결국 우아하게 커넥션 끊기는 무엇인가?

답은 HTTP 명세에서 정의된 바 없다.

일반적으로 우아한 커넥션 끊기를 구현하는 것은 애플리케이션 자신의 출력 채널을 먼저 끊고 다른 쪽의 기기의 출력 채널을 끊는 것을 기다리는 것이다.

즉, 양쪽에서 데이터 전송을 더이상 하지 않을 것이다라는 것을 보여주면 에러의 위험이 없이 온전히 종료된다.

하지만 다른 쪽의 기기가 절반 끊기를 했는지에 대한 보장도 없고, 했는지에 대해 검사도 못한다.

그래서 자신의 출력 채널을 끊은 후에도 데이터의 끝을 식별하기 위해 입력 채널에 대해 상태 검사를 주기적으로 해야하는 번거로움이 있다.

입력채널이 특정 타임아웃 시간 내에 끊어지지 않으면 애플리케이션은 리소스를 보호하기 위해 커넥션을 강제로 끊을 수 있다.
