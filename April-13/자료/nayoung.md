# 02. URL과 리소스

## 1. URI/URL/URN의 기본 개념
> **URI (Uniform Resource Identifier : 통합 자원 식별자)**
- 인터넷의 우편물 주소 : 웹 상에서 리소스(특정 자원)의 위치를 지정하고 고유하게 식별
- 종합적, 포괄적인 개념

![](https://velog.velcdn.com/images/mooongs/post/747ae8a2-e974-4dbf-89f3-469995a8371c/image.png)

> **URL (Uniform Resource Locator)**
- 리소스의 "위치"로 식별 (사람의 거주지 주소)
- 단일 방식의 작명 규칙으로, 리소스에 쉽게 접근하여 사용 및 공유 가능
- https://comic.naver.com/index


> **URN (Uniform Resource Names)**
- 리소스의 위치에 관계 없이 "이름" 만으로 식별 (사람의 주민등록번호)
- 여전히 실험 단계 중에 있으며 널리 채택되지 않음
- urn:ietf:rfc:2141

<br>

## 2. URL 문법 
![](https://velog.velcdn.com/images/mooongs/post/c2c3edfd-7003-4ce9-8b7f-0d1c90c38daf/image.png)

### 2-1. **스킴 ★**
- 어떤 프로토콜을 사용해서 리소스에 접근할지 (how)
- 프로토콜: 컴퓨터 네트워크에서 데이터를 교환하거나 전송하기 위한 규약들
- 대소문자 구분 X / 스킴명은 콜론(:)으로 구분

### 2-2. **사용자 이름과 비밀번호**
  > ftp://nayoung:my_passwd@ftp.prep.ai.mit.edu/pub/gnu

- 서버가 데이터에 접근할 때 요구 (ftp 서버에서 많이 사용)
- 이름:비밀번호 (위의 예시에서 nayoung:my_passwd)
- 쓰지 않으면 기본값 사용 (anonymous/비밀번호는 브라우저마다 상이)


### 2-3. **호스트와 포트 ★**
어플리케이션이 인터넷에 있는 리소스를 찾으려면, 리소스를 호스팅하고 있는 장비와 그 장비 내에서 리소스에 접근할 수 있는 서버의 위치를 알아야 한다. (where)
- 호스트 : 리소스를 가지고 있는 서버로 도메인명과 IP 사용 가능
- 포트 : 서버가 열어놓은 네트워크 관문
- TCP 프로토콜을 사용하는 http의 기본 포트 80 / https는 443

### 2-4. **경로 ★**
- 리소스의 위치 (what)
- /를 기준으로 여러 개의 경로 조각을 가질 수 있음

### 2-5. **파라미터 (Matrix Paramter)**
- 호스트, 경로 만으로 리소스를 찾지 못하는 경우가 많기 때문에 정확한 요청을 위해 파라미터 값이 필요
- 다른 요소들과 ';'로 구분되며 '이름=값' 쌍의 형식
  > ftp://prep.ai.mit.edu/pub/gnu;type=d


### 2-6. **질의 문자열 (Query String)**
- 웹 서버가 리소스 형식의 범위를 줄이기 위해 사용
- '&'로 구분되며 '이름=값' 쌍의 형식


### 2-7. 프래그먼트 
- 리소스의 일부분 가리킬 때
- 서버는 객체를 전체 단위로 전송하기 때문에 클라이언트에서 프래그먼트 작업 후 특정 문단을 보여줄 때 사용
<br>

## 3. 단축 URL
### 3-1. **상대 URL**
- 현재 기저 URL을 바탕으로 짧게 작성한 URL
- 스킴과 호스트가 기저 URL을 사용할 것이란 걸 예측하고 다시 절대 URL을 생성

    - 기저 URL : http://www.joes-hardware.com/tools.html
    - 상대 URL : ./hammers.html
    - 새로운 절대 URL : http://www.joes-hardware.com/hammers.html

### 3-2. **URL 확장**
- 호스트명 확장 (naver만 입력하면 www.naver.com)
- 히스토리 확장 (과거의 방문 기록 저장)
<br>

## 4. 리소스의 안전한 전송
**✅ 안정성이란?** 정보의 유실 없이 전송되는 것

### ❓ HOW 
이스케이프 문자 (퍼센트 문자)
: '%' 기호 + ASCII 코드로 표현되는 두 개의 16진수 숫자

![](https://velog.velcdn.com/images/mooongs/post/2d41886f-3d47-4b97-b67b-8ff95552f091/image.png)

  > ✅  **이스케이프** 
URL 설계자들이 URL에 담긴 정보를 안전히 전달하기 위해 안전하지 않은 문자를 안전한 문자로 인코딩 하기 위해 추가한 기능


### ❓ **WHEN**
- US-ASCII에서 사용이 금지된 문자
- 이미 의미가 선점되어 있는 문자들
 
![](https://velog.velcdn.com/images/mooongs/post/47579cb2-1152-45c8-9e89-0429c8a8cd25/image.png)
<br>

## 5. 스킴의 종류
![](https://velog.velcdn.com/images/mooongs/post/b52a0f8e-fc9e-4d6d-aa0f-d1fd2e1809b7/image.png)
<br>

## 6. URL 전망
- 👎🏻 URL의 단점 : 리소스의 위치가 변경되면 URL 또한 변경되어 객체 찾을 수 없음
- 🤷🏻‍♀️ 옵션 : URN / PURL(Persistent uniform resource locators)
- but 긴 표준화 작업, 인프라 변경 등 많은 문제 예상
