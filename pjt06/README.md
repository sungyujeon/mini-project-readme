# PJT06_사용자인증 기반 웹 페이지 구현

서울 1반 전선규



> 목표
>
> - CRUD 구현
> - django ORM을 이용한 데이터 처리
> - Authentication에 대한 이해
> - Database 1:N 이해

기본적으로 하나의 application(articles, posts, accounts 등)의 CRUD 구현을 연습하였다. 이를 구현하는 과정에서 V(view)에서 ORM을 이용하여 Database를 다루는 방법을 익혔다. 또한 데이터베이스를 구성하는 데 있어 관계형 데이터베이스를 만들었다.



## Check Point

##### 1. Authentication

django에서는 기본적으로 User 모델을 제공하여 사용자 인증에 대한 프레임워크를 제공한다. 이를 제공하지 않는다면 사용자에 대한 CRUD 구현과 back단에서의 유효성 검사, front에서의 입력 처리 등을 모두 서버 개발자가 구현해야 하지만, 이 프레임워크를 이용하여 기본적인 기능을 가져와 사용할 수 있었다. 이 중에서 CustomUser 모델을 만드는 것이 가장 헷갈리는 개념이었다. 

기본적으로 제공하는 User 클래스는 AbstractUser 클래스를 상속받는다. AbstractUser 클래스에서는 User 모델의 기본적인 정보들(username, firstname, email 등)을 정의하고 있고, 다시 상속받는 AbstractBaseUser에서 password에 대한 정의가 되어 있다. 나는 password에 대한 처리만 django에서 제공하는 기능을 사용하고 싶다면 AbstractBaseUser 모델을 상속받아 custom하고, 이외 정보들도 함께 사용하고 싶다면 AbstractUser 클래스를 상속 받아 사용하는 것으로 이해하였다. 

이 때, AbstractUser 클래스를 상속받아 새로운 CustomUser를 만들었다면, 추가적으로 구현해야 하는 클래스들이 있었다. 이번 프로젝트에서는 사용자의 회원가입에 대한 부분이었다. accounts app의 models.py에 CustomUser 클래스를 정의했다면, django에서 기본적으로 제공하는 UserCreationForm, UserChangeForm 등을 재정의해서 사용해야 한다.

이는 새롭게 정의한 CustomUser 모델이 기존의 User 모델과 일치하지 않으므로, 현재 사용하고 있는(active) 모델의 객체를 받아와야 하기 때문이다.

```python
# accounts/forms.py
import django.contrib.auth import get_user_model

class CustomCreationForm(UserCreationForm):
  
  class Meta:
    model = get_user_model()
    fields = '__all__'
```

다른 app의 models.py에서 user 객체를 참조(Foreign Key)하는 관계를 형성한다면, 이 때는 settings에서 설정한 AUTH_USER_MODEL을 인자로 넘겨주어야 하며, 이외의 모듈에서는 django.contrib.auth에서 제공하는 get_user_model() 함수를 사용하면 된다.

```python
# models.py
import django.conf import settings

class Review(models.Model):
  #...
  user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=CASCADE)
 
# accounts/forms.py
# 위 CustomCrationForm 클래스 참조
```

프로젝트 진행 시에 이 부분에 대해서 pair들과 함께 오류를 찾지 못하여 찾아보는 데 시간이 많이 걸렸다. 새롭게 Custom한 User 모델 객체를 참조하지 않고 기존의 auth에서 제공하는 User 모델을 참조하려고 하였기 때문이다. 위와 같이 현재 정의하고 있는 User 객체를 참조하여 문제를 해결하였다.



##### 2. Foreign key

comment를 create하는 view 함수를 정의하는 데 있어서 여러 테이블을 foreign key로 갖는 함수를 정의할 때 주의해야 할 점이 해당 객체를 모두 저장한 뒤에 save를 하여야 한다는 것이다. 아래의 코드처럼 comment를 하나 생성해주기 위해선 참조하고 있는 user, review 객체를 모두 할당하고 save를 해주어야 했다.

```python
@require_POST
def comment_create(request, review_pk):
    if request.user.is_authenticated:
        review = get_object_or_404(Review, pk=review_pk)
        
        form = CommentForm(request.POST)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.review = review      # review 객체를 할당
            comment.user = request.user  # user 객체를 할당
            comment.save()
            return redirect('community:detail', review.pk)
        
        context = {
            'form':form,
            'comment' : comment,
        }
        return render(request, 'community/detail.html', context)

    return redirect('accounts:login')
```



## 추가학습 필요 내용

디자인을 변경하기 위해서 bootstrap 클래스를 여러 태그에 적용시켜 보았다. 태그들이 html 문서에 가시적으로 작성되어 있을 때는 해당 태그에 어떤 bootstrap을 사용할건지에 대해 documentations을 참고하여 작성하면 되지만, form 태그의 경우 쉽지 않았다. 예를 들어 CustomUserCreationForm을 사용할 때 아이디와 패스워드의 제약사항을 정의하는 `help_text` `password_validators_help_text_html` 의 내용을 custom 하기 위해서는 어떤 방식으로 해야하는지 추가 조사가 필요했다.

help_text는 django에서 기본적으로 'Required. 150 characters or fewer. Letters, digits and @/./+/-/_ only.' 라고 제공하지만, 이를 '15자 이내의 문자, 숫자로 입력하시오.' 라는 텍스트로 바꾸기 위해서는 다음과 같이 수정해야 했다.

```python
class CustomUserCreationForm(UserCreationForm):
    
    class Meta:
        model = get_user_model()
        fields = UserCreationForm.Meta.fields
        help_texts = {'username': "15자 이내의 문자, 숫자로 입력하시오.",}
```



그런데!

username은 정상적으로 작동했지만, password는 작동하지 않았다. 아래와 같이 key 값으로 password, password1, password2 등을 아무리 넣어도 password에 대한 help text는 변하지 않았다. `help_texts = {'password1': 'password1 help text', 'password2': 'password2 help text'}`

이에 대해 검색한 결과 다음과 같은 해결책을 찾을 수 있었다.

```python
class CustomUserCreationForm(UserCreationForm):

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.fields['password1'].help_text = 'password1 help text'
        self.fields['password2'].help_text = 'password2 help text'
    
    class Meta:
        model = get_user_model()
        fields = UserCreationForm.Meta.fields
        help_texts = {'username': None,}
```

init 함수를 작성하여 self.fields의 password1,2의 help_text를 강제로 수정하는 방법이었는데 잘 동작하였다. 하지만 django에서 제공하는 html 태그로 해당 텍스트를 꾸미는 것은 더 이상 사용할 수 없었다.

결론적으로 django에서 제공하는 User 모델과 User 관련 form을 사용하는 것은 매우 편리하지만 이를 custom하기 위해서는 상당히 많은 노력과 검색, 시간이 필요할 것이라는 생각이 들었다. form을 customize 하는 것에 대해 더 알아보아야겠다.