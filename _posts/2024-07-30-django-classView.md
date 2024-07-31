---
layout: post
read_time: true
show_date: true
title: "django project의 클래스형 뷰 "
date: 2024-07-30 15:00:20 -0600
description: "클래스형 뷰, 제너릭형뷰 , "
image: https://images.velog.io/images/dldhk97/post/35a88d4c-ef9f-4d8e-a361-c6d53e4e2162/django-logo-negative.png
tags: 
    - coding
    - diary
   
author: soi

toc: no

---

### 클래스 뷰
클래스형뷰는 장고에서 요청을 처리하는 뷰를 클래스 형태로 정의하는 방법으로 기존에 사용했던 함수형 뷰보다 구조화되고 재사용이 가능하며 확장이 가능하다 

클래스 형뷰를 사용하기 위해서는 APIView를 import해 사용한다 
APIView를 상속받은 클래스를 만들고 안에 get, post의 함수를 생성해 method에 따른 로직을 지정해준다 
```python
class MovieList(APIView):
    def get(self, request):
        movies = Movie.objects.all()
        serializer = MovieSerializer(movies, many=True)
        return Response(serializer.data)
        
    def post(self, request):
        serializer = MovieSerializer(data=request.data)
        if serializer.is_valid(raise_exception=True): #유효하지 않으면 에러발생
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

     def put(self, request, pk):
        movie = self.get_object(pk) #객체를 받아온다 이건 url에서 받아온다
        serializer = MovieSerializer(movie, data=request.data, partial=True)
        if serializer.is_valid(raise_exception=True):
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        movie = self.get_object(pk)
        movie.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
이 클래스를 url에 연결할려면 .as_view()를 사용해 연결해야한다 
```python
urlpatterns = [
   path('movies', MovieList.as_view()),
]
```

### 제너릭 뷰
제너릭뷰는 REST framework에서 제공하는 클래스형 뷰로 공통적인 패턴과 기능을 쉽게 구현할 수 있게 도와준다 

crud를 포함한 요청 처리를 단순하고 효율적으로 구현할 수 있다 

- 제너릭 뷰 종류
1. ListAPIView: 객체 목록을 반환(GET)
2. RetrieveAPIView: 특정 객체의 세부 정보를 반환(GET)
3. CreateAPIView: 새로운 객체를 생성(POST)
4. UpdateAPIView: 기존 객체를 업데이트(PUT PATCH)
5. DestroyAPIView: 기존 객체를 삭제(DELETE)
6. ListCreateAPIView: 객체 목록을 반환하거나 새 객체를 생성(GET POST)
7. RetrieveUpdateAPIView: 특정 객체의 세부 정보를 반환하거나 업데이트(GET PUT PATCH)
8. RetrieveDestroyAPIView: 특정 객체의 세부 정보를 반환하거나 삭제(GET DELETE)
9. RetrieveUpdateDestroyAPIView: 특정 객체의 세부 정보를 반환 OR 업데이트 OR 삭제(GET PUT PATCH DELETE)

- get& create
```python 
from rest_framework.generics import ListCreateAPIView
class MovieList(ListCreateAPIView):
    queryset = Movie.objects.all() #필수옵션으로 돌려줄 객체지정
    serializer_class = MovieSerializer #생성시 사용랄 시리얼라이저 설정
```
이렇게 원모델을 바꾸면 참고하고 있던 모델울 리턴하는 뷰도 바꿔야 한다 

기존의 제너릭 뷰는 쿼리셋을 지정해야 하지만 이번에는 원모델의 값을 가져와 필터해야하기때문에 바로 지정이 불가능하다 
get_queryset() 함수를 사용해 지정해야한다 
```python
class ReviewList(ListCreateAPIView):
    serializer_class = ReviewSerializer

    def get_queryset(self): #get_queryset() 랑 queryset 둘중 하나만 정의하면됨
        movie = get_object_or_404(Movie, pk=self.kwargs.get('pk')) # url의 pr값은 self.kwargs.get로 접근 가능
        return Review.objects.filter(movie=movie)

    def perform_create(self, serializer):
        movie = get_object_or_404(Movie, pk=self.kwargs.get('pk'))
        serializer.save(movie=movie)

#시리얼라이저를 인수로 받아 serializer.save()를 실행 작성할 원모델의 아이디는 url에 있어 가져와 사용한다 
```

- update & delete
RetrieveUpdateDestroyAPIView를 사용한다  특정한 데이터를 조회, 수정, 삭제할 수 있다 
```python
class MovieDetail(RetrieveUpdateDestroyAPIView):
    queryset = Movie.objects.all()
    serializer_class = MovieSerializer
```
- 제너릭 뷰 옵션
queryset: ListAPIView는 쿼리셋을 직렬화 하고 RetrieveAPIView, UpdateAPIView, DestoryAPIView는 쿼리셋에서 특정한 객체를 가져온다

serializer_class: 직렬화나 역직렬화 해줄 시리얼라이저를 정하는 옵션으로 쿼리셋으로 가져온 데이터를 어떻게 직렬화랄지, 수정과 생성위해 어떻게 입력받을지 정할 수 있다 

-  RetrieveAPIView, UpdateAPIView, DeleteAPIView에서 사용되는 특정한 옵션
lookup_field : 쿼리셋에서 특정객체를 찾을때 어떤 필드를 기준으로 할지 정해준다 

lookup_url_kwarg: URL로부터 받아오는 변수명을 지정

### 제너릭 뷰를 이용한 페이지네이션
제너릭 뷰를 이용하면 페이지네이션을 쉽게 구현할 수 있다 

- 전역페이지네이션 설정
setting파일에 REST_FRAMEWORK변수를 만들고 안에 페이지네이션관련 내용을 작성한다 
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination', #페이지네이션을 사용하겠다 
    'PAGE_SIZE': 3,  #최대반환할객체
}
```
이후 요청을 보내면 기존과 달리 결과물이 3개씩 나오게 되고 cout라는 키를 통해 실제 존재하는 데이터의 개수, next라는 키를 통해 다음 데이터의 url(없을경우 null), previous이라는 키를 통해 이전 데이터의 url을 알려준다 
```json
{
    "count": 100,
    "next": "http://127.0.0.1:8000/movies/?page=2",
    "previous": null,
    "results": [
        {
            "id": 1,
            "name": "Movie 1",
            "opening_date": "2021-01-01",
            "running_time": 120,
            "overview": "Overview of Movie 1"
        },
        {
            "id": 2,
            "name": "Movie 2",
            "opening_date": "2021-01-02",
            "running_time": 115,
            "overview": "Overview of Movie 2"
        },
        {
            "id": 3,
            "name": "Movie 3",
            "opening_date": "2021-01-03",
            "running_time": 110,
            "overview": "Overview of Movie 3"
        }
    ]
}
```
- 개별페이지네이션
특정한 뷰에 페이지네이션을 별도로 설정할 수 있다 

먼저 PageNumberPagination를 상속받은 클래스를 생성하고 원하는 데이터 처리 개수를 설정한다 
```python
from rest_framework.pagination import PageNumberPagination
class MoviePageNumberPagination(PageNumberPagination):
    page_size = 2

class MovieList(ListCreateAPIView):
    queryset = Movie.objects.all()
    serializer_class = MovieSerializer
    pagination_class = MoviePageNumberPagination
```
만든 제너릭 뷰에   pagination_class 옵션을줘 적용한다 

### CORS
Cross-Origin Resource Sharing의 약자로 교차출처 리소스로 url의 프로토콜, 호스트, 포트를 합쳐서 출처라고 부른다 

cors는 출처가 다른 자원들을 공유한다는 뜻으로 서로 다른 출체어있는 자원에 접근할 수 있게 하는 개념이다 라지만 다른 출처에서 가져오는 것은 보안상의 문제가 있기 때문에 막아놨다 

이걸 해결하기 위해서는 서버에서 cors를 처리할 주소를 설정해놔야 한다 
```shell
pip install django-cors-headers
``` 
설치한 다음 setting 파일에 들어가 추가해줘야 한다 
```python
INSTALLED_APPS = [
    ...,
    'corsheaders',
]

MIDDLEWARE = [
    # 최상단에 작성
    'corsheaders.middleware.CorsMiddleware',
        ...,
]

CORS_ALLOWED_ORIGINS = [ 
    'http://localhost:3000', #허용할 출처 
]
```