# PJT04_프레임워크 기반 웹페이지 구현

> 목표
>
> - 데이터 CRUD(생성, 조회, 수정, 삭제) 기능 구현
> - DJango(Python Web Framework)를 통한 데이터 조작
> - ORM(Object Relational Mapping) 이해
> - Admin 페이지를 통한 관리



Django 프레임워크를 활용하여 영화 정보에 대한 간단한 CRUD 기능을 구현하는 프로젝트를 진행했다. HTML, CSS는 Bootstrap을 이용하였다.

이번 프로젝트는 처음으로 Pair Programming으로 진행하였다. 같은 프로젝트를 진행하는 데 있어 협업에 필요한 작업들을 추가적으로 배웠다. 가상환경 설정, DB data, Git에 대한 작업이 필요하였다.



## CRUD

1. C(create)

   > DB에 새로운 data를 생성하는 방법은 다음과 같이 세가지 방법이 있다.

   1) 인스턴스 생성 후 저장1

   ```python
   movie = Movie()
   movie.title = title
   movie.overview = overview
   movie.save()
   ```

   2) 인스턴스 생성 후 저장2

   ```python
   movie = Movie(title=title, overview=overview)
   movie.save()
   ```

   3) create 함수 사용

   ```python
   movie = Movie.objects.create(title=title, overview=overview)
   ```



2. R(read)

> DB에 저장된 데이터를 불러온다.

```python
def index(request):
    movies = Movie.objects.all()
    
    context = {
        'movies': movies,
    }

    return render(request, 'movies/index.html', context)
```

2) 특정 데이터 조회

`Movie.objects.get(<attribute>='<찾을 대상>')`



3. U(Update)

```python
def update(request, pk):  # DB 정보 수정
    movie = Movie.objects.get(pk=pk)

    title = request.POST.get('title')
    overview = request.POST.get('overview')
    poster_path = request.POST.get('poster_path')

    movie.title = title
    movie.overview = overview
    movie.poster_path = poster_path
    movie.save()
    
    return redirect('movies:detail', movie.pk)
```



4. D(Delete)

```python
def delete(request, pk):  # 데이터 삭제
    movie = Movie.objects.get(pk=pk)
    movie.delete()

    return redirect('movies:index')
```





## 가상환경 설정

>python에서 기본으로 제공하는 venv를 이용하여 가상환경을 설정할 수 있다.

`$ python -m venv .venv`  #.venv 폴더를 생성하여 가상환경 설정

`$ source venv/Scripts/activate`  # 해당 가상환경 활성화

`$ deactivate`  # 비활성화



> 가상환경에 설치된 pip 설정을 공유하기 위해 저장(개발환경 복구, 공유)

`$ pip freeze > requirements.txt`   # 최상위 폴더 내에 pip list를 저장

`$ pip install -r requirements.txt`  # 저장된 pip 설정을 설치



## DB data

> DB에 저장된 dummy data를 공유하기 위함. db 자체를 공유하면 충돌이 나기 때문에 dumpdata를 JSON 또는 XML 파일 등으로 변환하여 공유

- Save

`app/fixtures/app_name folder`  # dumpdata를 저장하기 위한 경로 설정

`python manage.py dumpdata movies.movie`  # dump data 생성하여 저장, app_name.table_name(소문자)

- Load

`python manage.py loaddata movies/movies.json`  # fixtures 경로에 있는 json 파일을 load



## Git

> 협업을 하기 위해 본인(A)과 상대방(B)의 git remote 주소를 local에서 한 번에 관리하여 B의 repository와 A의 repository 간의 자료 교환을 local에서 용이하게 만든다.
>
> `$ git remote add <name> <url>`

`git remote add origin <my url>`  # 내 remote 주소 등록

`git remote add pair <pair's url>`  # 상대방 remote 주소 추가 등록





## 마치며

위에서처럼 git, db, 개발환경 등의 sync를 상대방과 사전에 맞추어 수월하게 진행할 수 있었다. navigator와 driver로 나누어 코드를 작성하는 것도 큰 무리 없이 진행하였다. 충돌이 나는 상황에 대하여 해결하는 것은 어렵지 않을 수 있지만, 위와 같은 규칙들을 무시하고 작업하여 충돌의 빈도가 늘어난다면 상호 간 개발 진행에 피로도가 쌓일 것 같다. 나 혼자하는 개발이 아닌, 누군가와 같이 하는 개발을 하는 데에서 규칙을 만들고 지키는 것이 중요하다는 것을 다시 한 번 느낀다. 다만 향후 프로젝트를 진행할 때 어떤 충돌을 추가적으로 맞이할지 걱정이 든다. <small>(할 수 있겠지...?)</small>

