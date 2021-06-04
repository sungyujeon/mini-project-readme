# PJT08_데이터베이스 설계를 통한 REST API 설계

>목표
>
>- RESTful한 URL 설정
>- 데이터 CRUD API Server 제작
>- REST API 제작을 위한 Serializers.py views.py 모듈 구성

이번 관통 프로젝트에서는 기본적으로 CRUD를 구현한 API 서버를 제작하고, 사용자가 요청을 보내는 URL을 RESTful하게 구성하였다. API 서버 제작에 필요한 Serializer 설정을 추가적으로 하였다.



## 1. 기본 구성

> 추가 library, model 설정, url 설정

##### 1) 추가 library

- rest_framework

  RESTful API 개발을 위해 사용자의 요청 method를 세부적으로 구분하기 위해서 djangorestframework 라이브러리를 install하였다. 기존의 HTTP 요청 method는 'GET', 'POST'로 구분하였지만 이에 더해 'PUT', 'DELETE' 등의 method 분리가 필요하였다. 이를 가능하게 해주는 라이브러리를 아래와 같이 설치한다. 이후 installed_apps에 등록한다.

  ```python
  pip install djangorestframework
  ```

  ```python
  # pjt08/settings.py
  INSTALLED_APPS = [
    #...
    'rest_framework',
  ]
  ```

- django-seed

  DB에 dummy data를 삽입하기 위해 개별 데이터를 직접 입력해주는 것이 아니라 라이브러리를 이용하였다. 이를 이용하면 필요한 data를 자동으로 삽입할 수 있게 된다. 아래와 같이 install 해주고, dummy data를 삽입한다.

  ```python
  pip install django-seed
  ```

  ```python
  # pjt08/settings.py 등록
  INSTALLED_APPS = [
    #...
    'django_seed',
  ]
  ```

  ```python
  python manage.py seed movies --number=10
  ```

  

##### 2) model 설정

데이터의 구조를 작성하는 모델에서는 기존의 웹 프로젝트를 진행한 방식과 동일하게 구성해준다. API Server에서도 기존 DB에 저장되어 있는 데이터를 client에게 응답하는 방식으로 동작하기 때문에 별도의 설정은 필요없다.

##### 3) url 설정

API 서버에서 일반적으로 따르는 설정은 다음과 같다. url 주소는 'api/<버전>/'로 설정하여 추후 버전이 변경되었을 경우 url 주소의 변경 또는 추가를 일관성 있는 주소로 설정할 수 있게한다.

```python
# pjt08/settings.py
urlpatterns = [
		# ...
    path('api/v1/', include('movies.urls')),
]
```



## 2. RESTful URL

RESTful API는 REST 아키텍쳐의 제약 조건을 준수하는 app 프로그래밍 인터페이스를 뜻한다. REST(Representational State Transfer) 아키텍쳐는 client의 요청 method에 따라 요청이 분리되어 관리되는 client-server 아키텍쳐를 말한다. 

명세상에 제시된 API 서버를 개발하기 위해 다음과 같이 URL 주소를 구성하기로 하였다. RESTful한 API의 url을 구성하기 위해서 주소 내에 동작(verb)이 되는 요소는 제외하고, client의 요청 method에 따라 같은 url 요청으로 다른 동작을 하도록 했다.

##### 1:N 관계에서의 RESTful URL 구성

이 때, movie_id를 FK로 갖는 review와, review_id를 FK로 갖는 comment에 관한 요청을 처리할 때 url을 어떻게 구성하느냐에 대해 페어들과 여러 이야기를 나눌 수 있었다. 아래는 'movie/\<int:movie_pk>/review/\<int:review_pk>/'와 같이 구성한 예이다. 이 때 review 단일 데이터의 삭제, 수정을 하기 위해서는 review 테이블의 pk만으로 동작이 가능한 상황이었다. 하지만 우리의 개발 방향은 1) 단일 movie 정보가 존재하고 2) 이와 관계를 맺는 review 정보를 응답하고 싶은 것이었다. 즉, 단일 movie 정보와 review의 연관성에 더 중점을 두도록 개발을 하고 싶었다. 따라서 review 데이터의 수정, 삭제 시에 movie 테이블 정보가 불필요하였지만, url에 명시적으로 작성하여 사용자의 요청이 더 명확하게 드러날 수 있도록 하였다.

```python
# movies/urls.py
urlpatterns = [
    path('movie/', views.movie_list),  # 영화 전체 목록
    path('movie/<int:movie_pk>/', views.movie_detail), # 영화 단일목록
    path('movie/<int:movie_pk>/review/', views.review_list_or_create), # 리뷰 전체 목록, 리뷰 생성
    path('movie/<int:movie_pk>/review/<int:review_pk>/', views.review_detail),  # review 삭제, 수정
    path('movie/<int:movie_pk>/review/<int:review_pk>/comment/', views.comment_list_or_create),  # comment 전체 정보, 생성
]
```



## 3. CRUD API Server

##### serializers.py

아래는 ReviewSerializer의 예이다. rest_framework가 가지고 있는 serializers 모듈에서 ModelSerializer 클래스를 상속받는 ReviewSerializer 클래스를 생성한다. 이후 ModelForm과 비슷하게 Meta 클래스에 model과 fields 설정을 한다. 이 때 read_only_fields 리스트를 전달하여 server에서 처리를 해줘야 하는(movie_id를 FK로 가지기 때문에 이는 서버에서 처리해줌) field 설정을 한다.

추가적으로 comment_set은 해당 CommentSerializer에서 해당 review에 해당하는 queryset을 리스트 형태로 받고, 위 movie_id와 동일하게 read_only 속성을 True로 바꿔준다. comment_count는 새로운 IntegerField를 생성하는 것이므로, serializers가 가지고 있는 IntegerField 객체를 생성하여 전달하여 준다. 이 때 인자로 source와 read_only 정보를 함께 전달한다.

```python
# movie/serializers.py
from rest_framework import serializers
from .models import Review

# class CommentSerializer():
  # ...

class ReviewSerializer(serializers.ModelSerializer):
    comment_set = CommentSerializer(many=True, read_only=True)
    comment_count = serializers.IntegerField(source="comment_set.count", read_only=True)
    class Meta():
        model = Review
        fields = '__all__'
        # 사용자가 데이터를 전달할 때 인자로 같이 전달하지 않도록 read_only_fields를 작성해준다.
        # 안쓰게되면 movie_id를 사용자가 전달해줘야하는 상황이 발생한다.
        read_only_fields = ('movie',)
```



##### views.py

아래 #1, #2, #3은 각각 전체 영화 목록, 단일 영화 정보, review 목록(단일 movie에 대한), review 생성 처리를 하는 함수이다.

##### #1 전체 영화 목록

전체 영화 목록을 응답하기 위해서 movies queryset을 호출한다. 전체 queryset들을 list 형태로 받아오기 위해 get_list_or_404 메서드를 사용했다. 이후 MovieListSerializer에 해당 queryset 리스트를 전달하고, 여러개의 queryset이기 때문에 `many=True`속성을 함께 전달하였다. 이후 Response 객체에 serializer에 있는 queryset 데이터를 담아 반환하였다. 이 때, RESTful API 구성을 위해 api_view decorator를 사용해 사용자가 요청하는 method 리스트를 명시한다. 전체 영화목록은 GET 방식 요청이기 때문에 리스트에 'GET' 요청을 추가하였다.

##### #2 단일 영화 목록

단일 영화 목록은 해당 movie_pk 값을 갖는 queryset 객체 하나를 반환하기 때문에 get_object_or_404 메서드를 호출하였다. 이후 MovieSerializer에 해당 movie queryset을 인자로 전달하여 Response 객체에 담아 반환하였다.

##### #3 review 목록 조회, review 생성

review_list_or_create 함수는 review 목록 조회와 review 생성 두가지 동작을 하나의 url 주소로 처리한다. 해당 method는 GET과 POST 방식의 요청에 대해 분기하여 처리하였다. 

GET 방식의 목록 조회 요청은 단일 movie queryset을 참조하는 reviews 데이터를 movie.review_set.all()로 호출하였다. 이후 리스트 형태의 queryset을 ReviewSerializer의 인자로 전달하여 반환하였다.

POST 방식의 review 생성은 request에 담긴 data를 ReviewSeralizer에 전달하여 해당 데이터에 대한 유효성 검사를 시행하였다. 이 때 유효하지 않은 데이터가 요청되었을 경우에는 인자로 `raise_exception=True` 인자를 전달하여 처리하였다. 유효성 검사를 통과한 데이터는 serializer.save()를 통해 db에 저장하고, 이 때 read_only_fields에 선언한 FK인 movie_id 정보를 전달하기 위해 save메서드 인자로 movie 객체를 다음과 같이 함께 전달하였다. `serializer.save(movie=movie)`

```python
from django.shortcuts import get_list_or_404, get_object_or_404
from .models import Movie
from .serializers import MovieListSerializer, MovieSerializer, ReviewSerializer

from rest_framework.response import Response
from rest_framework.decorators import api_view

#1
@api_view(['GET'])
def movie_list(request):
    movies = get_list_or_404(Movie)
    serializer = MovieListSerializer(movies, many=True)   
    return Response(serializer.data)

#2
@api_view(['GET'])
def movie_detail(request, movie_pk):
    movie = get_object_or_404(Movie, pk=movie_pk)
    serializer = MovieSerializer(movie)
    return Response(serializer.data)

#3
@api_view(['GET', 'POST'])
def review_list_or_create(request, movie_pk):
    movie = get_object_or_404(Movie, pk=movie_pk)
    if request.method == 'GET':
        reviews = movie.review_set.all()    
        serializer = ReviewSerializer(reviews, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = ReviewSerializer(data=request.data)
        if serializer.is_valid(raise_exception=True):
            # 앞 movie = review에 들어있는 컬럼이름, 뒤 movie = 위에서 만든 pk값에 해당하는 movie객체
            serializer.save(movie=movie)
            return Response(serializer.data)
```



## 마치며

이번 관통 프로젝트에서 페어분들과 개발에 대한 대화를 가장 많이 한 듯 싶다. 이유는 REST API 개발에 있어서 개별 개발자의 개발 방향이 많이 반영되기 때문인 듯했다. 또한 URL 구성에 있어서도 client가 요청할 주소를 정하는 것에는 답이 없고 개발자가 정하기 나름이었기 때문이라고 생각한다. 

하지만 내가 생각하기에 API 서버를 개발하는 것도 결국에는 다른 개발자가 필요한 정보나 하나의 어플리케이션에 쉽게 접근하기 위해서라고 생각한다. 그러려면 사용자(개발자)가 다루기 쉬워야 하고, 이를 위해서는 일관된 규칙이 중요하다고 생각한다. 정해진 규약은 없지만 다수의 사용자가 쉽게 접근할 수 있도록 주소를 설정하고, 이미 정해진 규칙(예컨데 http method를 get, post, put, delete 등으로 나눈 것)을 지켜가며 개발하는 것이 위 목표를 달성하는 데 중요할 듯싶다. 

페어분들과 이야기를 많이 나누다보니 나도 공부가 되고 내가 헷갈리거나 편협하게 생각했던 부분에 대해서도 더 깊고 폭넓게 생각할 수 있는 기회가 되었다. 특히나 이번 프로젝트는 함께 개발하는 것의 즐거움을 더 많이 느낀 알찬 프로젝트였던 것 같다!