---
layout: post
read_time: true
show_date: true
title: "django project의 template과 view, model"
date: 2024-07-29 15:00:20 -0600
description: "template과 view,model"
image: https://images.velog.io/images/dldhk97/post/35a88d4c-ef9f-4d8e-a361-c6d53e4e2162/django-logo-negative.png
tags: 
    - coding
    - diary
   
author: soi

toc: no

---

### 정적파일 관리하기 
정적파일이란 html 파일을 제외하고 렌더링 하는 과정에서 필요한 추가적인 파일(css, js등)
장고는 이러한 정적파일을 static 디렉토리를 만들고 안에 app이름으로 된 정적폴더를 만든후 그곳에 정적 파일을 넣는다 

- 정적파일을 사용하도록 만들기 
진자처럼 html 파일 맨위에 템플릿 언어 사용하기
``` html
{ load static} <!-- % 넣어야 하는데 계속 오류나서 >

```
 
또한 파일의 경로를 템플릿 랭귀지를 사용해 \{%static 'appname/css/appname' %}\ 이렇게 바꿔준다 

- 디렉토리 구조
장고는 앱 안에 디렉토리를 만들고 다시 안에 app이름으로 디렉토리를 만든다 
그 이유는 사이트가 점점 복잡해지면 앱이 여러개가 되는데 이때 샌드위치 구조가 아니면 문제가 발생할 수 있다 
settings.py의 templates의 app_DIR이 있는데 여기가 기본값이 true이다 (각 앱에 템플릿이 있으니 거기서 찾으라는 의미)

찾는 순서는 installed_app에 등록된 순서대로 찾게 된다 그래서 이름이 같으면 안되는 것이다 

### 템플릿 언어 
html 문서를 작성할떄 프로그래밍 하듯 작성할 수 있게 해준다 

템플릿 언어는 크게 4가지로 나눌 수 있다 
- 템플릿 변수: 지정한 데이터로 변환 {{변수명}}
- 템플릿 태그: 템플릿 작성에 로직 사용 {%for %} 나 {%if %}
- 템플릿 필터: 템플릿 변수를 특정 형식으로 변환 {{변수명 | 필터}} 이 변수를 특정한 필터의 형태로변환
- 템플릿 주석: 템플릿 언어의 주석처리를 담당 {# 주석 #}

장고는 템플릿 상속을 사용해 여러 파일의 공통적인 부분을 모아 부모파일로 만들고 상속한다 
\{% block %}\, \{% extends %}\

부모템플릿은 바꾸고 싶은 부분을 블록으로 잡고 변경되지 않는 부분은 남겨준다 

\{extends 경로}\
이렇게 상속받고 블록을 열어서 안의 데이터를 채워주면 된다 

### 우아한 url
장고에서는 일정한 패턴을 가지고 입력하게 되는 url에 대해 다이나믹 url을 지원한다 
path('menu/<str:food>/', view.food_detail)
str:food 이 부분이 동적으로 바뀌는 url을 변수를 이용해 처리할 수 있게 해준다 

경로구분기호 / 를 제외한 모든 url을 문자열로 보고 food에 담아서 food_detail이라는 함수를 호출할때 인자로 넘겨줄 수 있다 (약간 js의 :path 느낌인듯)

이 str대신 0 또는 양의 정수와 일치하는지 체크하는 int, 하이픈 또는 언더바로 연결된 문자열과 일치하는지를 보는 slug등이 있음

넘어온 인자를 인수로 받고 리턴문에 context로 같이 넘겨주기(fast api 쓸때 진자 썻던것과 비슷하네)

### 에러페이지 처리하기
상ㅐ코드를 활용해 처리한다 
장고에서는 http404을 사용해 404 에러를 만들 수 있다 
raise를 사용해 파이썬에서 에러를 강제로 만들 수 있다 

### model 작성하기 
데이터베이스 설계를 먼저 한다 (이 부분은 개논물 모델링 하면 될거 같음)
class로 모델을 상속받아 구현해야 한다
각각의 데이터에 대한 형식을 필드를 사용한다 (charfiled, )
```python
from django.db import models

# Create your models here.
class Menu(models.Model):
    name = models.CharField(max_length=50)
    description = models.CharField(max_length=100)
    price = models.IntegerField()

    def __str__(self):
        return self.name

```
이렇게 모델이 만들어지면 장고에게 우리가 모델을 바꿨다고 알려줘야 한다 
```
python manage.py makemigrations 
python manage.py migrate
```
모델이 마이그레이션 되고 적용되게 된다

### 필드의 종류
- 필드 
데이터 테이블에서 컬럼을 말함

1. charfield: 인자로 max_length가 필수이며 제한된 길이의 문자열을 위한 필드

2. IntegerField: 정수값을 위한 필드 

3. BooleanField: 불린 값을 위한 필드 

4.  DateField(auto_now=False, auto_now_add=False): 파이썬의 datetime 객체 형태로 표시, auto_now는 해당 객체가 save될때마다 필드값을 지금으로 수정할것인지
auto_now_add 모델이 처음 생성될때 자동으로 필드 값을 설정한다 

5. BooleanField: Boolean 값을 위한 필드

6. EmailField: 이메일필드, 이용가능한 이메일인지 validation을 이용해 체크한다

### 마이그레이션
마이그레이션은 데이터베이스 변경사항을 저장해주는 목록으로 이 마이그레이션 목록을 실제 데이터베이스에 적용하는 것이 migrate 명령어이다 

우리가 모델객체를 파이썬으로 저장하면 django가 자동으로 ORM으로 바꾼 다음에 데이터베이스에 적용된다 

- 마이그레이션 명령어 

1. makemigrations
모델의 변경사항을 인식해서 새로운 마이그레이션을 만든다 만든 마이그레이션 파일은 각 앱의 migration 디렉토리에 저장된다 

```shell
python manage.py makemigrations
```

2. migrate
최신버전의 마이그레이션을데이터베이스에 반영하는 명령으로 전 마이그레이션으로 되돌리고 싶으면 
```shell
python manage.py migrate {appname} {되돌릴 번호}
```

3. showmigrations
모든 마이그레이션의 반영상태를 나타낸다 
```
python manage.py showmigrations

```

4. sqlmigrate
인수로 넘겨준 마이그레이션이 orm을 통해 변경된 sql문을 출력한다 
```shell
python manage.py sqlmigrate {앱 이름} {마이그레이션}
```

### CRUD
- Read
model.objects.all()
model.objects.all().values() 싱세내용 조회(조회하고 싶은게 따로 있다면 안에 이름을 적기)
model.objects.order_by(name) 인자의 값에 따라 오름차순(내림차순 하고 싶다면 인자 앞에 -붙이기)

조건을 걸어 한개만 가지고 오고 싶다면 get, 여러개 가지고 오고 싶다면 filter를 사용한다 
안의 조건은 필드명__조건키워드 =="조건" 형식이다 
- create
model.objects.create()

- update
먼저 get으로 데이터를 가져온 다음 그 값을 변수에 지정한다 그 후 값을 바꾸고 값을 save()를해 저장한다 

- delete
데이터를 get으로 가져온 다음 그 값을 변수에 지정한 후 .delete로 삭제한다 


### 쿼리셋
쿼리셋이란 데이터베이스와 소통할때 발생하는 자료형으로 리스트와 비슷하다 

데이터베이스에서 조회 해서 받아온 데이터는 쿼리셋에 들어가게되고 이 쿼리셋을 리스트처럼 사용할 수 있다

### 장고의 admin 도구
```
python manage.py createsuperuser
```
이렇게 관리자 계정을 만들고 개발서버의 admin으로 접속해 로그인하면 자동으로 만들어진 그룹과 유저 모델이 있다 

여기서 우리가 만든 모델을 이 관리자페이지에서 관리할 수 있도록 모델을 추가해줘야 한다

앱의 admin파일에 들어가 모델을 import하고 admin.site.register(이름)으로 등록한다 
(약간 프리즈마 스튜디오 같음)
