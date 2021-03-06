---
layout: post
title: "JWT 로그인 시스템에서의 자동 로그인"
date: 2020-01-12 21:00:00 +0900
categories: [development, django_rest]
show_sidebar: false
menubar: main_menu
permalink: '/category/django_rest/5/'
published: true
comment: true
---

# 자동 로그인의 기본적인 원리

자동 로그인을 하려면 기본적으로 클라이언트에게 무언가를 저장시켜줘야한다. 그리고 클라이언트가 저장한 값을 서버와 비교하여 신원을 확인하고 로그인 처리를 해준다.
여기서 문제는 클라이언트에게 무엇을 어디에 저장시키고, 언제 어떻게 그 무언가를 통해 로그인을 판단하느냐이다.

나는 개인적으로 글을 쓰는 시점에서 Django Rest Framework에서 jwt토큰을 이용하면서 어떻게 유효기간이 없는 자동로그인을 다루는지에 대해서는 찾지못해서, 내 나름대로 기존의 자동 로그인 이론과 이전에 설명했던 JWT토큰을 결합하여 방법을 구축해봤다. 이 부분에 정답은 없을 것이다. 내가 설명한 방식이 이미 보편적으로 쓰이는데, 내 지식의 범위가 좁아 찾지 못한 것일 수도 있고, 더 좋은 자동로그인 방식이 있을 수도 있다. 이번 글은 하나의 사례로써 봐주면 좋겠다.

# 사용하는 값들

기본적으로 로그인에서 쓰이는 jwt토큰으로는 access_token과 refresh_token이 있다.
그리고 자동로그인을 위해 쓰이는 토큰으로는 로그인마다 갱신되는 자동 로그인 토큰과 갱신되지 않고 변하지 않는 자동 로그인 키가 있다.

(작성 중)