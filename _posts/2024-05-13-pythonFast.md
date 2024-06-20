---
layout: post
read_time: true
show_date: true
title: "Python fastapi 설치 및 사용"
date: 2024-05-13 15:00:20 -0600
description: "Python fastapi 설치 및 사용"
image: https://velog.velcdn.com/images/soijeongg/post/a37a92a2-678e-4f28-9162-248b6abeeaa6/image.png
tags: 
    - coding
    - diary
   
author: soi

toc: no # leave empty or erase for no TOC
---

작년 12월에 진행한 챗봇프로젝트에 백프레임워크로 python의 fastapi를 사용했다 
[깃허브 링크](https://github.com/2023-AISCHOOL-IOTA/D.O)
사용한 이유는 다음과 같다 
1. fastapi는 비동기 웹프레임워크인 Starlette와 데이터검증 라이브러리인 Pydantic을 사용하기 때문에 다른 프레임워크들보다 빠르다 
챗봇은 사용자의 입력에 대한 빠른 대답 출력이 중요하기에 fastapi를 선택했다 

2. fastapi는 swagger을 기반으로 자동으로 api문서를 생성한다 
직접 문서를 만들어서 소통하는 것보다 더 정확하고 쉽게 문서를 만들고 소통할 수 있게 해준다 
![](https://velog.velcdn.com/images/soijeongg/post/4f474cd9-4b5d-4df2-a4e7-04a548262cf8/image.png)

3. fastapi는 비동기 프로그래밍을 지원하며, node나 go에 비견될 정도로 빠르다 
(개인적으로 node도 써봤지만 node는 python보다 더 사용하기가 어렵고 번거롭다고 느꼈다)

4. 공식문서에 써져있는 적은 버그와 높은 생산성, 쉬운 공식문서 
(이번에 나도 해서 쓴거 보면 그런거 같긴함)

또한 react+Fastapi+mongoDB 이렇게 세개를 같이 써서 FARM stack”이라고 함
gpt한테 물어봤더니 

FARM 스택은 데이터베이스, 백엔드 및 프론트엔드 기술을 포함하는 풀 스택 개발 스택의 하나입니다. FARM은 다음의 기술들을 나타냅니다:

FastAPI: FastAPI는 Python 기반의 웹 프레임워크로, 빠른 속도와 간편한 사용성을 제공합니다. 비동기 작업을 지원하고, OpenAPI 및 Swagger로 API 문서를 자동으로 생성하는 기능을 갖추고 있어 개발자들에게 매우 편리한 선택지입니다.
React: React는 페이스북에서 개발한 JavaScript 라이브러리로, 사용자 인터페이스를 만들기 위한 라이브러리입니다. 컴포넌트 기반의 접근 방식을 사용하여 재사용 가능하고 유지보수가 쉬운 UI를 구축할 수 있습니다.
MongoDB: MongoDB는 NoSQL 데이터베이스로, 문서 지향형 데이터 모델을 사용합니다. JSON과 유사한 BSON 형식의 문서를 저장하며, 유연하고 확장 가능한 데이터베이스 솔루션으로 널리 사용됩니다.
FARM 스택은 백엔드에서 FastAPI와 MongoDB를 사용하여 데이터를 처리하고, 프론트엔드에서 React를 사용하여 사용자 인터페이스를 구축합니다. 이러한 조합은 빠른 개발과 유연성을 제공하여 웹 애플리케이션을 구축하는 데 도움이 됩니다.
이렇게 나왔다 

### 시작
fastapi와 univorn을 설치 한다 
```shell
pip install fastapi 'uvicorn[standard]'
```
fastapi와 비동기 웹서버인 uvicorn을 설치한다 

- uvicorn이란?
ASGI 한 종류로 fast api와 함꼐 비동기적으로 작동함
Asynchronous Server Gateway Interface의 약자 비동기 웹 서버
![](https://velog.velcdn.com/images/soijeongg/post/718841c9-0c8f-49e3-b14d-97a983204b10/image.png)
fastapi는 혼자서 사용할 수 없고 웹서버가 있어야 한다 
uvicorn main:app --reload로 실행이 가능하다 기본 포트는 8000번인데 혹시 다른것과 같아서 바꿀려면 uvicorn app:app --host 0.0.0.0 --port 1234 이렇게 호스트를 0.0.0.0으로 하고 포트를 맘대로 설정할 수 있다 

### 기본예제 
```python

from fastapi import FastAPI
app = FastAPI()
@app.get("/hello")
def hello():
return {"message": "hello"}
```
app.py파일에 이렇게 코드를 작성해주고 위의 uvicorn main:app --reload를 사용해 실행시키면 
![](https://velog.velcdn.com/images/soijeongg/post/7e5fba5e-5e15-4a25-bf26-1e068e84d8f4/image.png)
이렇게 잘 나온다 
가장 기본적인 오직 서버를 열어서 보여주기만 하는 예제였다 

### response와 request
fastapi에서 response와 request를 사용하기 위해서는 fastapi모듈에서 가져와야 한다 
```python

from fastapi import FastAPI, Request, Response,HTTPException
app = FastAPI()

@app.get("/{error}")
def read_home(error:int,request: Request, response: Response):
    # 여기서 request와 response 객체 사용
    return {"message": "Hello World"}
    if error==1:
    	raise HTTPException(status_code=404, detail="에러메시지")
   return {"error":error}
```
HTTPException는 에러를 만들어낸다  const error = new Error 이렇게\

### pydantic
pydantic은 파이썬의 타입표기를 이용햐 데이터의 유효성을 검사하고 설정관리를 수행하는 라이브러리로 데이터 구조를 검증할 수 있다 
객체를 만들고 요청에서 사용하면 들어온 http요청중 같은 이름인것을 찾아 데이터를 처리한다 
```python

class login(BaseModel):
    id: str
    password: str
    
@router.post('/login')
def login(user: login, response: Response):
    expires = datetime.utcnow() + timedelta(hours=1)  
    user_in_db = collection_user.find_one({"id": user.id}) #로그인시 들어온 user라는 값에 있는 id를 가지고 일치하는게 있는지 검색
    password = user.password #비밀번호는 어차피 해싱 해야 되서 그냥 그대로 

# 비밀번호를 sha256으로 해싱 -> 해싱 시 16진수의 해시값이 됨
    hashed_password = hashlib.sha256(password.encode()).hexdigest()
    if user_in_db is None: #-> db #일치하는 값이 없다면(맞는 아이디조차 없다면)
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    #일부러 400에러 일으킴, 이거 숫자 바꾸면 진짜 웹이나 터미널에서 그 에러 났다고 나옴 신기
    elif user_in_db["password"] != hashed_password:  #아이디는 맞지만 저장된 비밀번호가 해시비밀번호와 다르다면
        raise HTTPException(status_code=400, detail="Incorrect username or password")# 오류
    
    token = create_jwt_token(user.id) #미들웨어py에 정의한 토큰 만드는 함수 사용해 id를 가지고 토큰을 만듬
    response.set_cookie(key="access_token", value=token, httponly=False, secure=False, expires =expires.timestamp() ) 
#   #쿠키에 저장 이름: access_token, 형식: tolen,httponly: 자바스크립트에서 쿠키 접근X 보안 강화를 위해  
#   #secure: 쿠키가 https에서만 전송 될 수 있게 이게 false면 그냥 http도 됨 
   
    return {"message": "로그인 성공"}
```
이게 실제로 내가 했던 코드인데 보면 로그인으로 정의해 가져오고 이걸 라우터에서 사용해 http요청중 이름이 로그인인거 받아서 사용한다 

### fastapi와 jinja2
fastapi에서 html페이지를 사용하기 위해 jinja2를 사용해야 한다 
jinja2란 파이썬에서 많이 사용되는템플릿 엔진 중 하나
Django에서 영감을 얻었지만 확장 됨

- 템플릿 엔진이란?
정적 템플릿(html)과 동적 데이터 결합하여 최종 출력을 생성 한다
정적인 HTML을 동적으로 바꿔준다
진자2가 해당 HTML문법을 템플릿으로 만들고 안의 파이썬 코드를 실행
템플릿을 체운 후 최종 HTML파일을 만듬
{% raw %}{{ 변수명}}{% endraw %}
{% raw %}{{% 파이썬 소스코드}}{% endraw %}여기에 끝에 for 나 if면 {% raw %}{{forend}}{% endraw %} 이런 식으로 붙은다
주석은{% raw %}{# #}{% endraw %}
이걸 사용해 html파일을 만든다 
그러니까 모듈화를 해서 조각조각 모을 수 있다 

진자2의 대 표적인 로직은 include, extends

include
include는 HTML을 부분부분 나누어 재사용 가능한 형태로 가공하는 기능이다
중복되는 부분 가지고 오는것

```html

<!--head.html-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world!</title>
</head>
```
이렇게 header 부분만 있는 파일읋 만듬

이걸 그대로 재사용 하고 싶다! 바꿀꺼 없다! 그냥 가지고 오고 싶다!

'{% include '파일명.html'%}' 이렇게 사용
간결하고 보기 쉽게 만들 수 있고 어디든지 가져다가 붙일 수 있어 유지보수에 좋다

extends
html을 상속받아 한 부분을 상황에 맞게 바꿔 쓸 수 있는 기능이다
그냥이 아니라 그 사이에 뭔가를 집어넣는거지
```html
{% raw %}
<!--{% block 이름 %}
상황별로 바꾸고 싶은 내용
{% endblock %} -->
{% endraw %}
```
이렇게 받은 후 동적으로 변할 부분을 바꾸기 위해 '{% block content %}'를 사용한다 

- 예시
```html
{% raw %}
<!-- header.html -->
<!DOCTYPE html>
<html lang="en">

<head>
    {% block head %}
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="icon" href="{{ url_for('static', path='/images/favicon.png') }}" type="image/x-icon">
    <link rel="stylesheet" href="{{ url_for('static', path= 'css/home.css')}}">
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css">
    <!--아이콘 넣기 위한 링크-->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>

    <title>{% block title %}home{% endblock %}</title>
    
    {% endblock %}
    {% endraw %}
</head>
```
이렇게 base파일을 만들고 블럭헤드를 만든다 
이걸 다른 곳에서 사용할때는 이렇게 사용한다 
```python
{% raw %}
{% extends 'header.html' %}
 
{% block title %}DOBOT{% endblock %}
<link rel="stylesheet" href="{{ url_for('static', path= 'css/home.css')}}">
{% block content %}
<main id="homemain" class="cont_main">
{% endraw %}
```

이렇게 서버를 열고 진자2를 사용해 html을 사용할 수 있다 
사용하는 js나 css는 매번 바뀌기 때문에 진자2에서 사용하는 문법인 url_for를 사용한다 
이 문법을 사용하면 정적파일 경로를 동적으로 바꿀 수 있다 
```html
{% raw %}
<link rel="stylesheet" href="{{ url_for('static', path= 'css/home.css')}}">
{% endraw %}
```
이렇게 사용하면 된다 