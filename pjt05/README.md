# PJT05_프레임워크 기반 웹페이지 구현

서울 1반 전선규



> 목표
>
> - CRUD 구현
> - ORM의 이해
> - ModelForm을 활용한 HTML과 사용자 요청 데이터 관리

이전 프로젝트에서는 Model을 정의하고 사용자의 요청에 따라 CRUD를 구현하는 내용을 진행했다. 이번에는 django에서 제공하는 ModelForm 클래스를 사용하여 더욱 간편하게 HTML form, 사용자의 요청에 대한 DB를 처리하는 내용을 연습하였다.

특히, ModelForm, Decorator, get_object_or_404를 중심으로 작성하였다.

## ModelForm

- django에서 제공하는 ModelForm은 이미 서버에서 정의한 Model의 정보를 참조하여 form을 구성하는 역할을 한다. 아래 코드 주석을 통하여 자세한 내용을 살펴보자.

```python
from django.forms import ModelForm  # froms 모듈의 ModelForm 클래스
from .models import Movie  # 참조할 Model 클래스

class MovieForm(ModelForm):  # ModelForm을 상속받는 클래스를 작성
  class Meta:  # 기반이 되는 데이터를 담는 Meta 클래스를 정의
    model = Movie  # model에는 참조할 Model 클래스를 담는다
    fields = '__all__'  # 해당 Model 클래스의 표현할 컬럼값을 명시
    
    # fiels에는 위와 같이 모든 정보를 가져올 수도 있지만 리스트 형태로 필요한 
    # 컬럼 속성만 가져올 수 있다.
    # fields = ['title', 'overview', 'poster_path']
```

```python
# views.py
# 기 정의한 Model의 정보를 담은 form 객체를 render하여 templates에 전달
form = MovieForm()

# 사용자로부터 받은 'POST' 요청 정보를 인자로 넘겨 저장 가능
form = MovieForm(request.POST)

# 특정 인스턴스에 요청 정보를 저장 가능
movie = Movie.objects.get(pk=pk)
form = MovieForm(request.POST, instance=movie)

# 유효성 검사
# 사용자로부터 받은 정보가 해당 Model의 데이터 타입에 유효한지 검사
form.is_valid()

# save()
# form 객체를 save하면 저장한 객체를 다시 return
movie = form.save()
```



- ModelForm은 form 객체를 넘기기 때문에 html 디자인 수정이 용이하지 않을 수 있다. 따라서 MovieForm 클래스의 Meta 클래스에서 widgets 속성을 사용해 키 값으로 각 컬럼값의 input tag의 스타일을 수정할 수 있다.

  ```python
  class MovieForm(ModelForm):
  	class Meta:
      model = Movie
      fields = ['title', 'overview']
      widgets = {
        'title': Textarea(attrs={'cols': 80, 'rows': 20}),
        'overview': Textarea(attrs={...})
      }
  ```



## Decorator

- django에서는 사용자의 요청을 분리하여 허용된 method 요청에 대해서만 처리를 하는 방식을 decorator 방식으로 지원한다. 다음과 같이 사용할 수 있다

  ```python
  from django.views.decorators.http import require_http_methods, require_POST, require_safe
  ```

  - `require_safe`  'GET'과 'HEAD' method 요청만을 처리

  - `require_POST` 'POST' 요청만을 처리

  - `require_http_methods` 리스트 형태의 인자로 전달한 요청을 처리

    > 허가되지 않은 method 요청은 HttpResponseNotAllowd, 405 status code를 반환한다.



## get_object_or_404

- 요청 method가 허가된 것이더라도 인자로 넘어온 값이 Model에 존재하지 않으면 'DoesNotExist' 에러가 발생한다. 즉, 해당 object를 가져오지 못하면 404 페이지를 리턴하는 것이다.

  ```python
  from django.shortcuts import get_object_or_404
  
  # delete 시에 해당 pk값이 존재하지 않는 것을 가정
  def delete(request, pk):
    movie = get_object_or_404(Movie, pk=pk)
    movie.delete()
   	return redirect('movies:index')
  	
    # get_object_or_404의 인자로
    # Model class, QuerySet 등을 전달
    # 키워드 인자로 get() 또는 filter() 할 수 있는 값을 전달
  ```



## Static / Media 설정

##### static 파일과 media upload path를 설정하기 위해 settings.py에 다음과 같이 작성

- STATIC

  ```python
  STATIC_URL = '/static/'  # static url 설정
  STATICFILES_DIRS = [BASE_DIR / 'pjt05' / 'static',]
  # app 폴더 내의 static 폴더가 아닌 전역에 설정할 시에 위와 같이 경로를 리스트 형식으로 추가하여 django가 static 폴더를 찾을 수 있도록 한다.
  ```

- MEDIA

  ```python
  MEDIA_ROOT = MEDIA_ROOT = BASE_DIR / 'media'
  MEDIA_URL = '/media/'
  # media_root는 사용자가 업로드한 파일을 실제 저장하는 경로를 설정
  # media_url은 사용자가 해당 미디어를 업로드하여 요청하기 위한 주소를 설정
  ```

  

## 마치며

django를 배울수록 헷갈리는 것이 많은 것 같다. 그 때마다 documentation을 참고하여 해결해나가는 방식을 배우는 것 같다. 수업에서 배운 내용 뿐만이 아니라 특정 기능이나 문법 등을 공식 문서를 참고하여 상세히 읽어보는 것이 특정 프레임워크를 배워가는 방법인 듯싶다. 영어로 되어 있어서 참고하기 귀찮고, 한글로된 자료만을 찾아보고 싶은 유혹이 많이 드는데 이를 극복하고 천천히, 꾸준히 자료를 정독하는 습관을 가져야겠다!!