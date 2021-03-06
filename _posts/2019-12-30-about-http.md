---
layout: post
title: "HTTP 통신이 뭘까?"
date: 2019-12-30 17:00:00 +0900
categories: [development, etc]
show_sidebar: false
published: true
comment: true
---


우리는 흔히 HTTP라는 글자를 많이본다. 비개발자입장에서도 주소를 입력할 때마다 보는 것이고, 개발자 입장에서도 HTTP라는 글자는 너무 많이본다. 개발자 입장에서 조금만 더 보자면, 블루투스나 적외선 같은 근거리 통신이 아닌, 서버 등과 같이 거리에 제약이 있으면 안되는 통신을 할 때 보면 항상 HTTP를 이용하고, HTTP 내에서도 GET, POST 등의 여러가지 방법을 나누어 처리하기도 하며, 헤더라는 것에 뭔가 인증키 같은 것을 넣기도 한다. 요즘은 개발 도구들이 비교적 쉽게 되어있어서 HTTP 통신이라는 것이 정확히 뭔지 몰라도 간단한 사이트 하나 정도는 만들 수 있기에, HTTP라는 것이 무엇인지에 대한 깊은 궁금증이 쉽게 유발되지 않는 것 같다(내 이야기다....). 

Django를 예로 들면, 클라이언트가 회원가입을 할 수 있는 사이트를 구현할 때 개발자는 대충 템플릿에 html코드로 form태그에 action에 url을 적당히 입력하고 method는 post로 한 다음, 여러 input태그들에 name을 넣고 submit타입의 버튼을 만든 뒤, views.py에서 url에 맞는 함수의 request.POST안에 있는 dict형태의 저장된 값을 불러와서 User를 create하면 된다.

이 과정에서 크게 HTTP가 뭔지는 알 필요가 없어보인다. GET, POST라는 method가 있다는 정도만 알면 작업이 가능하다(내가 그랬다....). 하지만 REST API나 기기에 제약받지 않는 HTTP통신을 해야할 경우에는 이렇게 얕은 지식으로는 무리가 있다.
그래서 이번 글에서는 나도 공부를 하는 겸, HTTP가 근본적으로 뭔지, 세상과 어떤 형태로 어떻게 통신을 하는지 가볍게 알아보겠다.

# HTTP가 뭐지? 하이퍼텍스트?
***

HTTP는 Hyper Text Transfer Protocol의 약자인데, 말 그대로 "하이퍼텍스트"라는 것을 전송하는 규약이다. 그럼 하이퍼텍스트는 뭘까? 직역하면 "초월적인 글"이다. 여기서 초월했다는 것은 기존의 글처럼 위에서 아래로의 서열로 정해진 것이 아닌, 순차가 없는 것을 의미한다. 순차가 없는 것은 무슨 말일까? 우리는 기본적으로 웹을 볼 때, 책을 읽듯이 순차적으로 글을 읽는 것이 아니라 "링크"라는 것을 통해서 여러가지 경우의 순서로 여러 컨텐츠들을 탐험한다. 이러한 방식으로 정보를 전달하는 텍스트를 "하이퍼 텍스트"라고 하는 것이다. 그래서 웹은 정보들이 이러한 하이퍼 텍스트들로 묶여있는 집합이라고 할 수 있다고 한다.

역사에 대해 간략히 얘기하자면 1965년에 테드 넬슨이라는 사람이 "하이퍼 텍스트"라는 말을 만들었고, 팀 버너스 리 라는 사람이 오리지널 HTTP를 만들었다고 한다. 최초로 문서화된 HTTP는 v0.9(1991년)이었다는데, 이 때는 헤더도 없었고 HTML형식의 텍스트만 주고 받았다. 메소드도 GET뿐.... 이후 1996년에 HTTP v1.0에서는 헤더 개념을 도입하였고, 상태코드도 전송하며 유연성과 확장성을 늘렸다. 이후 여러가지 모호성을 보완하여 몇달 뒤인 1997년에 HTTP v1.1을 표준프로토콜로 채택하였다. 이후 2015년에 더 진화한 HTTP/2가 발표되었다고 하는데, 이 이상의 역사에 대한 내용은 각자 찾아보기 바란다.

# HTTP의 기본적인 통신 방식
***

기본적으로 HTTP는 여러줄로 된 텍스트다. 여기에는 헤더, 바디 등의 구분을 공백으로 구분짓고, 헤더와 바디에 맞게 여러가지 정보들이 오고간다. 헤더는 주로 통신상태, 인증정보 등의 유저의 눈으로 직접 들어가지 않는 정보들이 들어가고, 바디에는 HTML나 JSON형식의 텍스트가 와서 유저의 눈에 직접 보이는 정보들이 주로 들어간다. 

통신을 할 때는 당연하게도 정보를 요청하는 쪽과 응답하는 쪽이 생기게 되는데, 이를 HTTP에서는 Request(요청)와 Response(응답)으로 나눈다. 클라이언트가 url을 설정하고 통신 메소드(GET, POST, PATCH 등)과 여러 정보들을 서버에 Request하면 서버에서는 정보를 처리하고 Response를 반환한다.

# HTTP 메시지의 형태
***

HTTP 메세지는 시작줄과 헤더와 바디로 구분되어있다. 시작줄은 항상 첫번째 한줄이며, 헤더와 바디는 공백한줄로 구분된다. 줄은 \<CR>\<LF>태그로 바뀐다(캐리지리턴, 라인피드).

Request의 HTTP 메세지 형태와 Response의 HTTP 메세지 형태는 조금 차이가 있는데, 각각 어떻게 생겼는지 아래 예시를 살펴보며 이야기하자.

```yml
POST /parties/45 HTTP/1.1       # 시작줄
HOST: localhost:8000            # 헤더 시작
Content-Type: application/json
Authorization: jwt <토큰>        # 헤더 끝   
                                # 공백
...                             # 바디내용
```

위와 같이 Request의 시작줄에는 GET, POST같은 메소드가 들어가고, HOST를 제외한 url, http버전이 들어간다.
그리고 헤더에는 인증정보나 콘텐츠타입 등의 부수적인 정보가 들어가고 공백 이후에 바디가 온다.


```yml
HTTP/1.1 200 OK                         # 시작줄
Date: Tue, 31 Dec 2019 06:51:30 GMT     # 헤더 시작
Server: WSGIServer/0.2 CPython/3.7.2
Content-Type: application/json
Vary: Accept
Allow: GET, PATCH, HEAD, OPTIONS
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Content-Length: 367                     # 헤더 끝
                                        # 공백
...                                     # 바디내용
```
Response의 시작줄에는 메소드나 url정보는 따로 없다. http의 버전이 먼저오고, 그다음은 숫자 세자리로 된 상태코드와 상태메시지가 온다. 그 이후에는 정보의 차이외에 형식적인 부분은 Request와 비슷하다.

이와 같이 Request와 Response 메시지의 결정적인 차이는 시작줄에 있다고 볼 수 있겠다.

이러한 형태의 HTTP 메시지들을 우리는 브라우저나 curl 등의 응용프로그램을 통해 필요한 정보를 보기쉽게 주고 받는 것이다.

# HTTP 상태 코드
***

공식 문서에 있는 자료를 가져왔다.
https://tools.ietf.org/html/rfc7231#section-6.1

## 1XX
통신 정보에 대한 응답이다. 자주 쓰진 않는 것 같다.

100 : Continue

101 : Switching Protocols

## 2XX
성공적인 응답들이다. 시스템이 정상적으로 작동했을 때 서버에서 보내준다.

200 : OK

201 : Created

202 : Accepted

203 : Non-Authoritative Information

204 : No Content

205 : Reset Content

206 : Partial Content

## 3XX
리다이렉션이 필요하다는 의미의 응답이다. 아직 공부가 부족하여 많이 보진 못했다.

300 : Multiple Choices

301 : Moved Permanently

302 : Found

303 : See Other          

304 : Not Modified       

305 : Use Proxy          

307 : Temporary Redirect 

## 4XX
클라이언트의 에러에 대한 코드이다. 개발자든 유저든 많이 보게 되는 번호들인 것 같다. 개발 시 꽤나 중요한 코드들이다.

400 : Bad Request        

401 : Unauthorized       

402 : Payment Required   

403 : Forbidden          

404 : Not Found          

405 : Method Not Allowed 

406 : Not Acceptable     

407 : Proxy Authentication Required

408 : Request Timeout    

409 : Conflict           

410 : Gone               

411 : Length Required    

412 : Precondition Failed

413 : Payload Too Large  

414 : URI Too Long       

415 : Unsupported Media Type

416 : Range Not Satisfiable

417 : Expectation Failed 

426 : Upgrade Required   

## 5XX
서버의 에러에 대한 코드. 요것은 유저입장에서 특히나 많이 보이면 안되는 코드들이다. 

500 : Internal Server Error

501 : Not Implemented    

502 : Bad Gateway        

503 : Service Unavailable

504 : Gateway Timeout    

505 : HTTP Version Not Supported  

# HTTP 메소드
***
메소드를 어떻게 선택하냐에 따라 데이터를 보낼 수 있는지, 서버에 어떤 행동을 요구하는지가 매우 달라진다. 특히 GET, POST, PATCH, DELETE는 각각 CRUD(Create, Read, Update, Delete)에 해당하므로 꼭 숙지해두자.

### GET
기본적으로 서버에 정보를 요청할 때 쓰는 것이다. 정보를 받는 것만 가능하다. 일반적으로 주소창에 URL을 입력하고 엔터를 누르면 GET방식의 호출을 한다. 기본값이라고 생각하면 될듯.

CRUD에서 일반적으로 READ를 담당한다.

### POST
정보를 전송할 수 있는 메소드이다. 정보는 body에 넣어서 보낸다. Content-Type헤더에 따라서 body에 들어가는 정보의 형태가 달라진다.

일반적으로 HTML form의 정보를 주고받는데 쓰이고, json형식으로 정보를 주고받을 때도 자주 쓰인다.

CRUD에서 일반적으로 CREATE를 담당한다.

### PATCH
정보를 전송하여 리소스의 일부만 수정할 수 있는 메소드이다. PUT메소드도 비슷한 용도로 많이 쓰이는데, P
UT은 리소스의 정보 전체를 바꿔야하는 메소드인 반면, PATCH는 리소스에 대해 제시된 정보만을 부분적으로 수정할 수 있다. 정보의 형태는 POST와 같이 body에 들어간다.

CRUD에서 일반적으로 UPDATE를 담당한다.

### DELETE
특정 리소스를 삭제하는 메소드이다. 

당연하게도 CRUD에서 DELETE를 담당한다.<br><br>

#### 아래부터는 CRUD에 해당하는 메소드는 아니다.

### HEAD
GET과 비슷하지만, body는 가져오지 않고 header의 정보만 가져온다. 정보 전체를 받기 전에 받을 정보의 크기가 어떤지 어떠한 정보인지 간단히 알기 위해 사용하는 메소드이다.

### OPTION
헤더에 허용하는 메소드를 표현하는 Allow항목을 포함하여 결과를 불러온다. body도 불러온다.
```yml
Allow: GET, POST, HEAD, OPTIONS
```
요런 형태의 헤더를 꼭 포함한다.

<br>
#### 이 외의 메소드는 따로 설명하지 않겠다....

<br><br>
---
***
일단 내가 자주 찾아보게되는 정보 위주로 적었다. HTTP 통신에 대해서는 훨씬 더 디테일하고 많은 내용들이 있다.

하지만 일단 여기 적혀있는 것은(역사빼고...) 기본적으로 알고 있어야하는 정보들이라고 생각한다.

상태코드의 경우에는 구구단처럼 암기할 필요까진 없다고 생각하지만, 최소한 2xx, 4xx, 5xx 가 무엇을 의미하는 지는 무조건 알아야한다고 생각한다.