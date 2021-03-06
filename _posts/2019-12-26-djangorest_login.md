---
layout: post
title: "로그인(JWT토큰 방식)"
date: 2020-12-26 21:00:00 +0900
categories: [development, django_rest]
show_sidebar: false
menubar: main_menu
permalink: '/category/django_rest/4/'
published: true
comment: true
---


# 기존 Django의 로그인 인증방식

기존의 Django에서는 세션과 쿠키를 이용하여 로그인 여부를 관리한다.
조금 더 자세하게 설명하자면, 인증의 과정은 아래와 같다.

1. 클라이언트(유저)가 서버에 아이디와 비밀번호 등의 로그인에 필요한 정보를 보낸다.
2. 서버는 받은 정보가 회원DB에 있는 정보인지 비교한다.
3. 만약 유효한 회원정보라면, 세션 저장소(Django에선 DB)에 세션정보를 생성한다.
4. 생성된 세션정보의 세션ID를 발급하여 클라이언트의 쿠키에 저장한다.
5. 클라이언트는 발급받는 세션ID를 쿠키에 넣은 상태로 서버에 데이터를 요청한다.
6. 서버는 쿠키 안에 있는 세션ID를 읽어 세션 저장소에서 해당 세션ID의 만기일자를 확인하고, 만기된 세션이 아니라면 요청한 작업을 진행하고 데이터를 반환한다.

이렇게 세션과 쿠키를 이용한 방식은 사용자의 민감한정보(개인정보)를 직접적으로 계속 주고 받지 않고
세션ID라는 정보만 주고받으며 비교적 안전한 통신을 한다.


하지만 조금 아쉬운 점은 세션저장소가 따로 필요하며, 통신을 할 때마다 세션저장소에 접근해야한다는 점이 있고,
보안적인 점에서 아쉬운 점은 HTTP요청을 해커가 가로채서 세션ID를 뺏길 가능성이 있다는 것이다.


물론 이와 같은 일을 방지하기 위해 HTTPS를 이용하는 것으로 알고 있다. HTTPS와 세션만료시간을 이용하면 어느정도 방지가 가능하긴 하다.

# 토큰 인증 방식(JWT)

토큰 인증 방식 또한 세션과 쿠키를 이용하는 방식과 같이 많이 쓰인다. Json Web Token의 약자라고 한다.

이 방식은 클라이언트(유저)입장에서는 세션, 쿠키 방식과 크게 다르진 않다. 세션, 쿠키방식이 세션ID를 헤더(쿠키)에 넣고 데이터를 주고 받는 방식이라면 JWT는 세션ID대신 토큰을 넣는 것 뿐이기 때문이다.

그럼 이 둘의 차이가 뭘까?

가장 큰 차이는 헤더에 포함되는 정보의 차이라고 볼 수 있겠다. 세션ID는 세션을 식별할 수 있는 그저 임의의 긴 문자열일 뿐인데 비해,
JWT는 정보를 암호화할 방식(Header), 서버에서 보내는 데이터(Payload), Base64방식으로 인코딩된 Header와 Payload에 secret key를 더한 값(Verify Signature)이 합쳐진 문자열이다.

https://jwt.io/ 에서 JWT방식이 어떻게 정보를 암호화하는지 확인 할 수 있다.

기본적으로 <Header의 16진수 인코딩>.<Payload의 16진수 인코딩>.<Verify Signature의 base64기반 인코딩>로 이루진다.
여기서 Header와 Payload는 고도의 암호화 방식이 아니기에 쉽게 디코딩이 될 수 있는 정보다. 그러므로 Payload에 비밀번호등의 중요한 정보는 들어가면 안된다고 한다.

하지만 Verify Signature는 secret key가 없으면 복호화하지 못한다.
그래서 해커가 만약 Payload의 정보를 위조해도 Verify Signature의 정보를 위조할 수 없는 한, 토큰을 조작할 수는 없게된다.

정리하면 JWT의 인증 과정은 아래와 같다.

1. 클라이언트(유저)가 서버에 아이디와 비밀번호 등의 로그인에 필요한 정보를 보낸다.
2. 서버는 받은 정보가 회원DB에 있는 정보인지 비교한다.
3. 만약 유효한 회원정보라면, JWT방식으로 토큰을 생성하여 클라이언트에게 보낸다.
4. 클라이언트는 발급받는 토큰을 헤더에 넣고 서버에 데이터를 요청한다.
5. 서버는 토큰을 secret key를 이용하여 토큰을 복호화하고 요청받은 작업을 수행한 후 데이터를 반환한다.

세션과 쿠키를 이용하는 방식과 비교해보면 JWT 인증 과정에서는 토큰에 정보가 들어있기 때문에 별도의 세션 저장소가 필요없다. 그러므로 프로그램의 확장성이 높아지고, 유지보수가 용이해진다.

또한 "구글 아이디로 로그인", "Facebook아이디로 로그인"등의 OAuth의 인증방식이 모두 JWT를 이용하므로 서비스의 확장성도 좋아진다.

그래서 일반적인 REST API들은 대부분 JWT방식의 인증을 사용하는 것으로 알고 있다.

Django Rest Framework에서도 JWT 인증방식에 대한 도입이 쉽게 모듈이 만들어져있다. 이제부터 JWT 인증방식으로 로그인을 관리해보자.

# PIP에서 모듈 설치

```yml
pip install djangorestframework-jwt
```

# settings.py 설정

아래와 같이 변수들을 설정한다.
```yml
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
            'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
    ),
    ...
}
JWT_AUTH = {
    'JWT_SECRET_KEY': SECRET_KEY, #Verify Signature의 secret key
    'JWT_ALGORITHM': 'HS256', #암호화에 쓰이는 알고리즘
    'JWT_ALLOW_REFRESH': True, #갱신 허용 여부
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=7), #기본 토큰 만료 제한
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(days=28), #토큰 갱신 제한
}
```

# urls.py 설정

원하는 url에 token을 생성하는 기능을 배치하기 위해 urls.py에 들어가서 아래와 같이 jwt모듈의 기능을 임포트한다.
어느 경로의 urls.py에 설정할지는 본인의 기호에 따라 알아서한다.

```yml
from rest_framework_jwt.views import obtain_jwt_token, verify_jwt_token, refresh_jwt_token
```
obtain_jwt_token : username과 password로 토큰을 생성하고 반환한다.
verify_jwt_token : 토큰의 유효성을 검증한다.
refresh_jwt_token : 토큰을 갱신한다.

아래와 같이 기능들을 할당한다.
```yml
urlpatterns = [
    ...,
    path('api/token', obtain_jwt_token),
    path('api/token/refresh', refresh_jwt_token),
    path('api/token/verify', verify_jwt_token)
]
```

이렇게 설정하면 api/token 경로로 username과 password를 POST로 보내면 token이 반환된다.
갱신 설정을 했다면, 받은 token을 api/token/refresh 경로로 POST로 보내면 만료가 갱신된 token이 반환된다.
토큰을 api/token/verify 경로로 POST로 보내면 유효한 경우 token을 그대로 반환하고 유효하지 않으면 에러메세지를 보낸다.

DjangoRestFramework-jwt 모듈에 대한 더 정확하고 자세한 정보는 아래 링크를 확인하도록 하자. 공식 문서다.
https://jpadilla.github.io/django-rest-framework-jwt/