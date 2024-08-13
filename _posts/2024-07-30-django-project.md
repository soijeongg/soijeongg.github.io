---
layout: post
read_time: true
show_date: true
title: "django project 정리"
date: 2024-08-02 15:00:20 -0600
description: "django이용한 rest api project"
image: https://images.velog.io/images/dldhk97/post/35a88d4c-ef9f-4d8e-a361-c6d53e4e2162/django-logo-negative.png
tags: 
    - coding
    - diary
   
author: soi

toc: no

---

### 프로젝트 시작
먼저 프로젝트 구조 설정 -> django-admin startproject myproject

그 후 안에서 각각의 기능을 담고 있는 app을 만든다 이 app은 꼭 하나의 프로젝트에만 들어가는게 아니라 여러 프로젝트에 들어갈 수 있다 그렇기 떄문에 한가지 app은 유연하고 다른 사람에게 배포가 가능하게 만들어야 한다

```
python manage.py startapp users
```
만들고 난 후 에는 프로젝트의 설정파일의 INSTALLED_APPS에 추가한다
이렇게 되면 이 프로젝트에서 이 app을 쓰겠다는 말

### 데이터베이스 연결
각 앱의 모델파일을 열고 모델을 정의한다


django는 django.contrib.auth 앱을 통해 기본 User 모델을 제공한다

```python
# Create your models here.
class Post(models.Model) :
    author = models.ForeignKey('auth.User', on_delete=models.CASCADE)
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

```

데이터베이스 연결은 setting 파일에서 정의한다 만약 mysql을 이용해 연결하려면 mysqlclient 패키지를 설치한다(다른 데이터베이스를 사용하려면 그걸 깔면 된다(psycopg2-binary 등))

``` python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'your_database_name',      # RDS에서 설정한 데이터베이스 이름
        'USER': 'your_database_user',      # RDS에서 설정한 마스터 사용자 이름
        'PASSWORD': 'your_database_password',  # RDS에서 설정한 마스터 사용자 비밀번호
        'HOST': 'your_rds_endpoint',       # RDS 인스턴스 엔드포인트
        'PORT': 'your_rds_port',           # RDS 인스턴스 포트 (기본값은 3306)
    }
}
```
이렇게 설정 후 django 모델을 데이터베이스에 연결하기 위해 마이그레이션을 한다

이때 먼저 데이터베이스가 만들어져 있어야 하니 vscode 익스텐션에서 mysql을 깔고 연결한다

그 후 create Datebase로 데이터베이스를 만든다

```
python manage.py makemigrations #마이그레이션 파일을 생성한다
python manage.py migrate  #마이그레이션을 실행한다
```
앱의 django의 admin에 들어가 유저모델을 관리자 페이지에 등록한다  이런 관리자페이지에 로그인 하기 위해서는 슈퍼 유저를 생성해야한다 
```
python manage.py createsuperuser
```
이 명령어를 사용해 슈퍼유저를 등록하면 된다

### 쿼리셋과 시리얼라이저

쿼리셋은 django의 ORM의 핵심으로 데이터베이스에서 데이터를 조회, 필터링, 정렬, 조작하는데 사용한다

쿼리셋을 통해 데이터베이스와 상호작용할 수 있으며 쿼리를 직접 안쓰고 데이터베이스 작업을 할 수 있다(ORM)

쿼리셋을 이용해 셀에서 데이터베이스작업이 제대로 되는지 확인하고 테스트 할 수 있다

시리얼라이저는 djano rest framwork에서 데이터를 json이나 xml으로 변환하거나 반대로 바꾸는 것에 사용된다  이는 모델 인스턴스를 직렬화 하거나 역직렬화 할 수 있다

이는 백엔드와 프론트엔드의 소통을 위해 데이터 형식을 통일하는데 사용된다

DRF(djangorestframework)는 요청에 따라 처리된 데이터를 프론트엔드에 전달한다

serializers.py를 만들고 시리얼라이저를 정의한다

``` python

class PostSerializer(serializers.Serializer):
      id = serializers.IntegerField()
      author = serializers.CharField()
      title = serializers.CharField()
      content = serializers.CharField()
      created_at = serializers.DateTimeField()
      updated_at = serializers.DateTimeField()
    def create(self, validated_data): #validated_data는 유효성검사 맟핀 데이터라는 의미로 위의 데이터가 딕셔너리 형태로 전달
        return Post.objects.create(**validated_data)

#이 시리얼 라이저를 간단하게 사용하기 위해 ModelSerializer를 사용한다
#기존에는 create, update를 정의해야힜지만 시리얼라이저 안에 존재하는 meta클래스 선언하고
#사용할 모델과 필드를 사용할지 정의하는 방식을 사용한다
class PostSerializer(serializers.ModelSerializer):
    author = serializers.CharField(source='author.username', read_only=True)

    class Meta:
        model = Post
        fields = ['id', 'author', 'title', 'content', 'created_at', 'updated_at']
```


### CRUD 뷰 만들기 
각 앱의 view파일은 계층 패턴의 service와 같이 로직을 처리하는 역할을 한다

django에서 rest api를 만들기 위해 djangorestframework(DRF)를 렌더링한다

pip install djangorestframework

그 후 INSTALLED_APPS에 rest_framework를 추가한다

view 파일에 가서 crud를 한다 

``` python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .models import Post
from rest_framework import status
from django.shortcuts import get_object_or_404
from .serializer import PostSerializer
# Create your views here.

@api_view(['GET', 'POST', 'DELETE'])
def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk) #고유기본기

    if request.method == 'GET':
       serializer = PostSerializer(post)
       return Response(serializer.data, status=status.HTTP_200_DK)
    
    if request.method =="POST":
          serializer = PostSerializer(data=request.data)
          if serializer.is_vaild():
              serializer.save()
              return Response(serializer.data, status=status.HTTP_201_CREATED)
          
    if request.method =="DELETE":
        post.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

이런 함수형 뷰를 클래스 형태로 정의하는 클래스 뷰는 함수혈뷰보다 구조화되고 재사용이 가능하며 확장이 가능하다

클래스 형 뷰는 APIView를 import하고 이를 상속받아 클래스를 만들고 각 method의 함수를 생성해 로직을 지정한다

``` python
class MovieList(APIView):
    def get(self, request):
        post = Movie.objects.all()
        serializer = PostSerializer(movies, many=True)
        return Response(serializer.data)
        
    def post(self, request):
        serializer = PostSerializer(data=request.data)
        if serializer.is_valid(raise_exception=True): #유효하지 않으면 에러발생
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

이를 제너릭 뷰를 사용해 공통적인 패턴과 기능을 쉽게 구현할 수 있게 도와준다
``` Python
from rest_framework.generics import ListCreateAPIView, RetrieveUpdateDestroyAPIView
from .models import Post
from .serializer import PostSerializer
# Create your views here.

from rest_framework.generics import ListCreateAPIView, RetrieveUpdateDestroyAPIView
from .models import Post
from rest_framework.permissions import IsAuthenticated
from .serializer import PostSerializer
# Create your views here.

class PostList(ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticated] 

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

class PostDetail(RetrieveUpdateDestroyAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticated] 
#자동으로 pk를 추출해 특정 객체 값을 검색한다
```

그 후 url에 연결해준다

함수형 기반 뷰와 다르게 클래스형기본뷰는 명시적인 리턴을 정의해줄 필요가 없다

``` python
from django.contrib import admin
from django.urls import path
from users.views import PostDetail, PostList

urlpatterns = [
    path('admin/', admin.site.urls),
    path('posts/<int:pk>/', PostDetail.as_view(), name='post-detail'),
    path('posts', PostList.as_view(), name ='postList')
]
```

### 관계맥지 
모델에서는  ForeignKey필드를 사용해 관계를 맺을 수 있다

그 후 시리얼라이저에서는 참조한 모델의 readOnly를 true로 한다

뷰에서 가져올떄 필터를 걸어 참조한 객체가 같은것만을 가져오고 그것을 시리얼라이저에 넣거 직렬화 한뒤 리턴한다