---
layout: post
read_time: true
show_date: true
title: "django project의 template과 view, model"
date: 2024-07-30 15:00:20 -0600
description: "template과 view,model"
image: https://images.velog.io/images/dldhk97/post/35a88d4c-ef9f-4d8e-a361-c6d53e4e2162/django-logo-negative.png
tags: 
    - coding
    - diary
   
author: soi

toc: no

---

### DRF란
DRF란 django REST Framwork
프론트엔드는 데이터를 백엔드에 요청하고 백엔드는 요청에 맞게 처리하여 데이터를 반환한다 
백엔드와 프론트엔드의 통신을 위해 데이터 형식을 통일해야한다 
이 과정을 **직렬화와 역직렬화라고 한다**

- 직렬화
서버에서 파이썬 객체로 저장된 데이터를 제이슨 형태로 바꿔주는 것

- 역직렬화 
제이슨 형태의 데이터를 파이썬 객체로 바꿔주는것

DRF는 응답으로 html과 css를 반환하는 django와 다르게 요청에 따라 처리된 데이터만 프론트엔드에 전달한다 

DRF를 사용하기 위해서는 settings파일에 들어가 rest_framework를 추가한다 

### 시리얼라이저 
django에서 직렬화를 수행하는 도구로 serializers를 상속받아 사용한다 

```python

from rest_framework import serializers
from .models import Movie

class MovieSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    name = serializers.CharField()
    opening_date = serializers.DateField()
    running_time = serializers.IntegerField()
    overview = serializers.CharField()
```
생성할 클래스에 시리얼라이즈 클래스를 상속시키고 필드를 정의해줘야한다 
이때 중요한점은 **사용할 필드 이름은 꼭 모델에서 사용할 필드의 이름과 일치시켜야 한다**

- 모델과 뷰를 연결
 api_view를 사용해 http method가 뭐가 들어오는지 결정한다 

 ```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

from .models import Movie
from .serializers import MovieSerializer

@api_view(['GET', 'POST'])
def movie_list(request):
    if request.method == 'GET':
        movies = Movie.objects.all()
        serializer = MovieSerializer(movies, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)
    elif request.method == 'POST':
        data = request.data #post 요청으로 전달된데이터
        serializer = MovieSerializer(data=data)
        if serializer.is_valid(): #데이터가 유효한지 검사한다 
            serializer.save() #통과하면 저장한다
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

그동안 템플릿을 보여주기 위해 썻던 render가 아니라 api_view, Response를 사용한다 

@api_view는 데코레이터 함수로 기존 함수를 수정하지 않고 추가 로직을 넣고 싶을떄 사용한다 
여기에는 기존의 함수에 api_view기능을 추가하기 위해 사용했다 

api_view(['GET'])는 이 함수가 get메소드만을 허용하는 api를 제공한다는 것을 표시한다 

그 후 시리얼라이저와 모델을 import하고 가져와 모델에서 값을 가져온다 
모든 객체를 가져와 직렬화해주기 위해 시리얼라이저에 넣는다 파라미터인 many는 여러 데이터를 가져왔기 때문에 true
그후 시리얼라이저를 통해 파이썬 딕셔너리 형태로 바뀐 값을 response로 리턴한다 

### 시리얼라이저
시리얼라이저 파일에서 각각의 코드를 살펴보자 

``` python
class MovieSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True) #조회때만 사용하고 싶을때 
    naㅋme = serializers.CharField()
    opening_date = serializers.DateField()
    running_time = serializers.Intege3rField()
    overview = serializers.CharField()

    def create(self, validated_data): #validated_data는 유효성검사 맟핀 데이터라는 의미로 위의 데이터가 딕셔너리 형태로 전달
        return Movie.objects.create(**validated_data)
# 언패킹(**)이란 리스트나 딕셔너리 형태로 감싸져 있는 값을 풀어서 사용하는 것으로 감싸져 있는 값을 풀어서 사용할 수 있다 
```
### 정리하기 
그러니까 파이썬 객체로 만든 모델을 제이슨으로 만들기 위해 시리얼라이저를 한다 
이 시리얼 라이저는 api와 클라이언트간의 통신을 위해 사용한다 

먼저 모델을 만들고 그 모델을 makemigration, migrate로 적용한다  

디장고에서 들어온 url를 보고 어디의 로직을 실행할지 결정하는 프로젝트의 url에 들어가 이 path로 오면 include를 사용해 특정앱의 url폴더를 살펴보라고 한다 

특정앱의 url 파일에서는 그 특정앱의 뷰의 함수를 사용한다고 설정한다 

함수에 api_view를 사용해 http method가 뭘떄 특정 함수를 실행한다


### 업데이트 
업데이트도 create와 마찬가지로 api_view에 put을 추가하고 http method가 put일때 할 로직을 설정한다 

데이터를 수정하기 위해 시리얼라이저에 update 함수를 정의한다 
수정할 데이터인 instance와 검증된 데이터인 validated_data를 받고 수정하려는 객체인 instance에 validated_data를 넣으면 데이터가 수정된다 

```python
def update(self, instance, validated_data):
        instance.name = validated_data.get('name', instance.name)
        instance.opening_date = validated_data.get('opening_date', instance.opening_date)
        instance.running_time = validated_data.get('running_time', instance.running_time)
        instance.overview = validated_data.get('overview', instance.overview)
        instance.save()
        return instance
```

get은 딕셔너리에 키에 맞는 값이 존재하면 데이터를 반환한다 
값이 존재한다면 수정 요청한 값이 없다면 기존의 필드값을 넣고 없다면 기존 필드 값인 instance.name을 넣고 수정한다 

뷰에 들어가 함수를 만든다

```python
@api_view(['GET', 'PUT', 'DELETE'])
def movie_detail(request, pk):
    movie = get_object_or_404(Movie, pk=pk) # 객체 받아오기 
    if request.method == 'GET':
        serializer = MovieSerializer(movie)
        return Response(serializer.data, status=status.HTTP_200_OK)
    elif request.method =="PUT":
        serializer = MovieSerializer(movie, data=request.data)
        if serializer.is_valid(): #검증
            serializer.save()    #저장하기
            return Response(serializer.data, status=status.HTTP_200_OK)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

- **시리얼라이저에서 create, update 함수를 만드는데 어디서 사용하는걸까?**
이게 계속 이해가 안되서 찾아봤었다 뷰의 함수를 보면 결국은 .save를 통해 업데이트와 create를 둘다 한다고 하는데 어떻게 하는걸까? 어디서 create와 update를 쓰는걸까??

 REST framework에서 save는 시리얼라이저의 create와 업데이트를 자동으로 호출한다 

 이때 save는 시리얼라이저가 이미 유효성 검사를 통과한 후 호출이 되고 인스턴스가 존재하지 않는 경우는 create로 인스턴스가 있을때는 update를 호출한다 

 ### delete
 delete가 들어오면 찾은 객체들을 delete 한 후 204를 리턴한다 
```python 
elif request.method == 'DELETE':
        movie.delete()
        return Response(status = status.HTTP_204_NO_CONTENT)
```
## 시리얼라이저

### 시리얼라이저 필드 정리
1. charField
문자열 데이터를 받는 필드로 모델이 CharField나 TextField를 받으면 이 필드를 사용한다 
최대 길이를 설정하는 max_length, 최소길이를 설정하는 min_length, 빈데이터 허용하는 allow_blank 옵션이 존재한다 

2. IntegerField
정수 데이터를 받는 필드로 모델이 IntegerField, SmallIntegerField이면 사용한다 
max_length와 min_length 둘다 필수는 아니지만 설정할 수 있다 

3. DateField
날씨 데이터를 받는 필드로 모델의 DateField와 매치된다 
format옵션은 데이터의 포멧을 나타내는데 기본은 -으로 연결되어있는 방식이고 다른 방식이 하고 싶으면 지정해줘야 한다 

4. DateTimeField
날짜와 시간이 모두 있는 필드로 모델의 DateTimeField와 매치 된다 

5. SerializerMethodField
사용자가 정의한 함수를 통해 직렬화 과정에서 새로운 값을 생성 할 수 있는 필드다 
데이터생성시에는 사용할 수 없는 readOnly 필드로 이름을 지정하는 method_name옵션을 설정하지 않으면 get_변수명으로 지정된다 

다른 필드의 값을 사용해 가져와 새로운 값을 만들떄 사용한다 

### 시리얼라이저 옵션정리

1. read_only: 생성시 입력하지 않지만 조회시에는 보여준다(기본키에 사용)
2. write_only: 데이터를 생성할때는 입력하지만 조회시에는 안보여야 하는 필드에서 사용한다 
3. required: 필수적으로 입력해야하는 것을 정의하는 값 기본값은 true(auto_now_add를 모델에서 쓰면 자동으로 생성되기 떄문에 시리얼라이저에서는 required를 false로 만든다 )

### 모델 시리얼라이저 
기존에는 시리얼라이저를 만든후 create, update함수를 정의해줘야했지만 이를 간단하게 사용하기 위해 ModelSerializer를 사용할 수 있다 

모델 시리얼라이저는 시리얼라이저 안에 존재하는 클래스로 내부에서 meta클래스를 선언하고 사용할 모델과 필드를 사용할지 정의하는 방식으로 사용한다 

```python
class MovieSerializer(serializers.ModelSerializer):
    class Meta:
        model = Movie
        fields = ['id', 'name', 'opening_date', 'running_time', 'overview']
```
모델시리얼라이저는 어떤 모델을 사용할지 model을 지정해줘야 하고 어떤 필드를 사용할지 작성하는 field를 지정해줘야 한다 (필드는 조건부 옵션으로 exclude를 사용하지 않으면 필수)
모델시리얼라이저는 데이터베이스에서 작성해주는 필드에 자동으로 read_only옵션을 넣는다

- 모델시리얼라이저의 옵션
exclude: exclude는 field와의 정반대로 어떤 필드를 제외할 지 선택하는 옵션이다 

read_only_fields: 리드온리를 자동으로 해주지만 필드가 아닌 자동으로 생성되지 않는 필드에 선택적으로 readOnly를 사용할 수 있다 

extra_kwargs: 다양한 필드에 여러 옵션을 추가해야하는 경우 사용할 수 있다 

### validators
시리얼라이저의 모든 필드에는 validators 옵션을 적용해 검사를 할 수 있다
시리얼라이저에 적용할 수도 있고 직접 검사 함수를 커스텀헤서 사용할 수도 있다 

```python

from django.core.validators import MaxLengthValidator, MinLengthValidator
from rest_framework.serializers import ValidationError
def overview_validator(value):
    if value > 300:
        raise ValidationError('소개 문구는 최대 300자 이하로 작성해야 합니다.')
    elif value < 10:
        raise ValidationError('소개 문구는 최소 10자 이상으로 작성해야 합니다.')
    return value

class MovieSerializer(serializers.ModelSerializer):
    overview = serializers.CharField(validators=[MinLengthValidator(limit_value=10), MaxLengthValidator(limit_value=300)])
     overview2 = serializers.CharField(validators=[overview_validator])
```
UniqueValidator와 UniqueTogetherValidator를 사용해 유일성을 확인할 수도 있다

- validate_[필드명]()
하나의 필드에서만 유효성 검사를 진행할려면 validate_[필드명]() 함수를 사용할 수 읶다 안에서 is_valid()함수가 실행될때 자동으로 실행되며 안에서 만든 로직에따라 검사를 수행하고 통과하지 못하면 에러를 반환한다 

```python
# ...
from rest_framework.serializers import ValidationError

class MovieSerializer(serializers.ModelSerializer):
    # ...
    
    def validate_overview(self, value):
        if 10 <= len(value) and len(value) <= 300:
            return value
        raise ValidationError('영화 소개는 10자 이상, 300자 이하로 작성해주세요.')
```
여러개의 필드를 한꺼번에 검사할려면 validate()를 사용한다  하나의 값을 사용하는게 아니라 사용하는 모든 필드의 값을 받는다 

### 관계맺기 
모델의 ForeignKey필드를 사용해 특정모델을 참조할 수 있다 

```python
class Review(models.Model):
    movie = models.ForeignKey(Movie, on_delete=models.CASCADE)
    username = models.CharField(max_length=30)
    star = models.IntegerField()
    comment = models.CharField(max_length=100)
    created = models.DateTimeField(auto_now_add=True)

```
직렬화하기 위해 시리얼라이저 생성하는데 따로 명시해야하는건 없고 참조한 모델을 readOnly를 true로 잡는다  

뷰에서 가져올떄 필터를 걸어 참조한 객체가 같은것만을 가져오고 그것을 시리얼라이저에 넣거 직렬화 한뒤 리턴한다 

### 역관계
원모델에서 관계설정해 처리할 수 있다 이 경우 데이터를 더 쉽게 관리할 수 있다 
1. ManyToManyField
다대다 관계를 정의한다 

```python
class Movie(models.Model):
    name = models.CharField(max_length=255)
    opening_date = models.DateField()
    running_time = models.IntegerField()
    overview = models.TextField()

    def __str__(self):
        return self.name

class Actor(models.Model):
    name = models.CharField(max_length=255)
    movies = models.ManyToManyField(Movie, related_name='actors') # 역참조

    def __str__(self):
        return self.name
```

2. OneToOneField
일대일관계 

```python
class Movie(models.Model):
    name = models.CharField(max_length=255)
    opening_date = models.DateField()
    running_time = models.IntegerField()
    overview = models.TextField()

    def __str__(self):
        return self.name

class Director(models.Model):
    name = models.CharField(max_length=255)
    movie = models.OneToOneField(Movie, related_name='director', on_delete=models.CASCADE) #역참조

    def __str__(self):
        return self.name
```
근데 OneToManyField는 없어서 ForeignKey를 사용해 역참조를 해야한다 