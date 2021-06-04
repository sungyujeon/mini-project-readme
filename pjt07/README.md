# PJT07_관계형 데이터베이스 설계

> 목표
>
> - CRUD 구현
> - django ORM을 이용한 데이터 처리
> - Authentication에 대한 이해
> - Database 1:N, M:N 이해

관계형 데이터베이스 설계에 중점을 두어 프로젝트를 진행하였다. 저번 프로젝트까지 1:N 관계의 모델링을 했다면, 이번 프로젝트에서는 M:N 관계의 모델링을 통해 follow, like 기능을 구현할 수 있었다. 추가적으로 hashtag 기능을 구현하고, 중복 문자에 대한 hashtag 에러를 바로잡았다.



## 1. like 기능 구현

like 기능을 구현하기 위해서는 like를 누르는 사용자와 like의 대상이 되는 게시물 간에 M:N 관계의 모델링을 할 필요가 있었다. 여러 사람이 한 게시물에 좋아요를 누를 수 있고, 한 사람이 여러 개의 게시물에 좋아요를 누를 수 있기 때문이다. 이 때 User 모델에 like 필드를 만들 수도 있고, Review 모델에 like 필드를 만들 수도 있지만, 한 게시물에 여러 사람이 좋아요를 누를 수 있다는 의미를 더 살리기 위해 Review 모델에 필드를 선언했다. Review를 작성하는 user를 참조하기 위해서 이미 user 필드가 선언되어 있었기 때문에 related_name을 넣어 참조 시 이름이 충돌되지 않게 하였다.

```python
# community/models.py

class Review(models.Model):
  # ...
  user_likes = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name='user_likes')
```



like를 누르거나 total likes를 확인할 수 있는 detail.html 페이지에서 다음과 같이 작성하였다. like를 누르는 사람은 1) 로그인이 되어 있어야 하고, 2) 로그인 한 사용자와 좋아요를 누르는 대상이 되는 사용자가 같지 않아야 했다. 이 때, 해당 review에 이미 like를 누른 사용자는 like를 취소할 수 있는 버튼이 보여지며, 누르지 않은 사용자는 누를 수 있는 버튼이 보여지게 구성하였다.

```html
<!-- community/templates/community/detail.html -->

{% if request.user.is_authenticated %}
	{% if request.user != review.user %}
		{% if request.user in review.user_likes.all %}
			<a href="{% url 'community:like' review.pk %}"><i class="fas fa-heart"></i></a>
		{% else %}
			<a href="{% url 'community:like' review.pk %}"><i class="far fa-heart"></i></a>
		{% endif %}
	{% endif %}
{% endif %}
<p>{{ review.user_likes.all|length }} likes</p>
```



views.py 함수에서 like 기능은 다음과 같이 구현하였다. like 함수 내에서 1) 사용자가 로그인되어 있는지 확인하고, 2-1) 해당 사용자가 해당 리뷰의 좋아요를 눌렀으면 remove를, 2-2) 누르지 않았으면 add 해주도록 하였다.

```python
# community/views.py

def like(request, review_pk):
    if request.user.is_authenticated:
        review = get_object_or_404(Review, pk=review_pk)

        if request.user in review.user_likes.all():
            review.review_like.remove(request.user)
        else:
            review.review_like.add(request.user)
        
        return redirect('community:detail', review_pk)
    
    return redirect('accounts:login')
```



## 2. follow 기능 구현

follow 기능을 구현하기 위해서는 팔로우를 하는 사람과 받는 사람이 모두 필요하다. 이 때도 M:N 관계가 설정될 수 있는데, 특이한 점은 모두 User 테이블을 참조한다는 것이다. 이 때 django ORM에서는 ManyToManyField의 인자로 'self'를 넘겨준다. 필드의 이름은 followings가 될 수도, followers가 될 수도 있다. 자기 자신으로 M:N 관계를 설정할 시에는 A 사용자가 B 사용자를 참조하게 될 때 자동적으로 B 사용자도 A사용자를 참조하게 설정이 된다. 따라서 이 설정을 바꾸기 위해 'symmetrical=False' 인자를 넘겨준다. 아래와 같이 followings로 만들 경우 accounts_user_followings 테이블이 만들어지고, from_user_id와 to_user_id 컬럼이 만들어진다. 따라서 'A 사용자'가 팔로우 하고 있는 User들을 조회하고 싶을 때는 AUser.followings.all()로 접근한다. 만약 related_name 인자를 설정해주지 않는다면, 팔로워들을 조회하기 위해 AUser.followings_set.all()로 접근하겠지만, 통일시켜주기 위해서 related_name='followers'도 함께 작성하였다.

```python
# accounts/models.py

class User(AbstractUser):
    followings = models.ManyToManyField('self', symmetrical=False, related_name='followers')
```



해당 내용을 출력하는 profile.html 페이지에서 1) 현재 로그인한 유저와 팔로우를 하려는 유저가 다를 경우에만 팔로우 버튼이 보일 수 있도록 하였고, 2-1) 로그인 한 유저가 이미 해당 유저를 팔로우 했으면 언팔로우 버튼을, 2-2) 팔로우 하지 않았으면 팔로우 버튼을 보여줄 수 있도록 하였다.

```html
<!-- accounts/templates/accounts/profile.html -->
{% if request.user != user_info %}
  {% if request.user in user_info.followers.all %}
    <a href="{% url 'accounts:follow' user_info.username %}"><i class="fas fa-star"></i></a>
  {% else %}
    <a href="{% url 'accounts:follow' user_info.username %}"><i class="far fa-star"></i></a>
  {% endif %}
{% endif %}
<p>following : {{ user_info.followings.count }}명</p>
<p>follower : {{ user_info.followers.count }}명</p>
```



view 함수에서는 현재 로그인한 유저와 팔로우를 하려는 유저의 객체를 받아서 처리하였다. 현재 로그인한 user의 변수명을 'me'로, 팔로우 하려는 user의 변수명을 'you'로 하여서 혼란을 줄였다.

```python
# accounts/views.py
def follow(request, user_id):
    me = request.user
    you = get_object_or_404(get_user_model(), username=user_id)

    if me == you:
        return redirect('accounts:profile', you.username)
    
    if me in you.followers.all():
        you.followers.remove(me)
    else:
        you.followers.add(me)

    return redirect('accounts:profile', you.username)
```



## 3. hashtag 기능 구현

pair인 반장님과 함께 hashtag 기능을 구현해보기로 하였다. 이 때, 기존에 해왔던 hashtag는 중복 문자가 존재할 경우 전체 hashtag 문자열이 a 태그로 감싸여지지 않는 문제가 있었다. 그래서 이를 해결하기 위해 별도의 로직을 만들었다. 우선 hashtag 기능을 구현하기 위한 기본 구성은 다음과 같다.

##### model 설정

hashtag를 만들기 위해서 우선 모델링을 한다. Hashtag는 하나의 게시물에 여러개가 달릴 수 있고, 여러 개의 게시물은 또 여러개의 hashtag들을 가질 수 있기 때문에 M:N 관계로 설정할 수 있었다. 우선 Hashtag 테이블은 개별 hashtag명을 담는 content 필드를 선언하고, 같은 hashtag에 대해서는 중복해서 들어가지 않도록 'unique=True' 인자를 넣었다. 해당 hashtag들이 들어가는 Review 모델에서 M:N 관계를 맺고, hashtags 테이블을 만들어 Hashtag와 Review 테이블의 PK를 참조할 수 있게 하였다. 이 때, 하나의 review에서 hashtags로 접근하기 위해 review.hashtags를 쓰는 것처럼 hashtag에서 review로 접근하는 이름을 통일하기 위해 related_name을 'reviews'로 설정하였다. 또한 'blank=True'를 하여 해당 필드가 비어 있어도 가능하게 하였다.

```python
# community/models.py

class Hashtag(models.Model):
    content = models.CharField(max_length=100, unique=True)

class Review(models.Model):]
  	# ...
    hashtags = models.ManyToManyField(Hashtag, related_name='reviews', blank=True)
```



##### view 함수 설정

첫째로, 해당 review 글이 작성될 때 해시태그가 들어가는 것이기 때문에, create 함수에서 로직을 처리하여야 했다. POST에 담겨 있는 작성된 review.content에서 '#'으로 시작하는 단어를 가져와 Hashtag 테이블에 넣어주었다. 이 때, Hashtag 테이블은 unique 설정이 되어 있으므로, 같은 값이 중복해서 들어갈 경우 에러가 난다. 따라서 Hashtag 테이블의 Hashtag.objects.get_or_create(content='<해당하는 단어>')를 사용하였다. hashtag를 사용했을 때 해당 hashtag를 갖는 게시물(review) queryset을 가져오기 위해 hashtag 함수도 만들었다.

```python
# @login_required
# def create(request):
#     if request.method == 'POST':
#         form = ReviewForm(request.POST)
#         if form.is_valid():
#             review = form.save(commit=False)
#             review.user = request.user
#             review.save()
            
            for word in review.content.split():  # hashtag 처리 로직
                if word.startswith('#'):
                    hashtag, created = Hashtag.objects.get_or_create(content=word)
                    review.hashtags.add(hashtag)

    #         return redirect('community:detail', review.pk)
    # else:
    #     form = ReviewForm()
    # context = {
    #     'form': form
    # }
    # return render(request, 'community/create.html', context)
```

##### html 출력

html에서 출력하여 줄 때, 위 view 함수까지만 설정한다면 해당 review에 담긴 content 내용이 그대로 출력된다. 하지만 원하는 기능은 해당 hashtag를 a 태그로 감싸서 해당 해시태그를 가진 게시물로 이동할 수 있도록 처리하는 것이다. 이를 위해서 templatetags를 사용해야 했다. 내장되어 있는 django template filter가 아닌 위 로직을 처리할 수 있는 filter를 만들기 위해   다음과 같은 폴더 구조를 갖는 파일을 생성한다.

app

. . . templatetags

. . . . . . \__init__.py

. . . . . . my_filter.py

해당 필더를 사용하기 위해 my_filter 파일에서 다음과 같이 설정하여 준다. template.Library()에 register 해주고, 이를 decorator로 함수를 감싸주어야 한다. 이 때 주의사항은 서버를 껐다 켜야 적용된다는 점이다.

```python
# community/templatetags/my_filter.py

from django import template
from community.models import Hashtag

register = template.Library()

@register.filter
def text2link(review):
  pass
```

이렇게 하면 html 페이지에서 다음과 같이 custom filter를 사용할 수 있다. detail 페이지에서, review 쿼리셋을 가져오고, text2link 함수에서 해당 쿼리셋의 content를 조작하여 a 태그로 감싸게 한다. safe filter는 해당 script를 html이 인식할 수 있도록 만들어주는 것이다. html tag와 같은 스크립트는 보안상의 이유로 django에서 텍스트 데이터로 자동으로 렌더하는데, safe filter를 사용하면 text로 바꾸지 않고 그대로 script로 인식할 수 있게 해준다.

```html
<p>{{ review|text2link|safe }}</p>
```

text2link 로직은 다음과 같이 만들었다. 

1) review 객체에 있는 content를 content에 담고, 이를 split하여 새로운 content_list를 만든다.

2) review 객체로 역참조하여 해당 pk를 갖는 hashtags들을 모두 가져온다.

3) 태그들을 모두 돌면서, 해당 태그가 content_list에 담긴 내용들 중에 일치하는 것이 있는지 확인한다. 이 작업을 반복한다. 일치하는 문자가 있으면 hash2link 함수를 이용해 a 태그로 감싸준다. 마지막으로 해당 content를 join 함수를 통해 text 데이터로 반환한다.

```python
@register.filter
def text2link(review):
    content = review.content
    tags = review.hashtags.all()
    content_list = content.split()  #list
    
    for tag in tags:
        for i in range(len(content_list)):
            char = content_list[i]
            if char == tag.content:
                content_list[i] = hash2link(char, tag.pk)
    
    content = ' '.join(content_list)
    
    return content


def hash2link(_char, _tag_pk):
    return f'<a href="/community/hashtag/{_tag_pk}/">{_char}</a>'
```

마지막으로 생각해볼 것이, 이 방법이 과연 효율적인가에 대한 고민이었다. 반장님과 이야기 하였을 때, 모든 tag들을 돌면서, 띄어쓰기로 구분된 content의 단어들을 매번 돌면서 a 태그로 바꾸어주는 것이 비효율적이라는 생각이 들었다. 만약 tag가 N개 있고, content의 단어들이 M개일 때, O(N * M)의 시간복잡도가 걸릴 것 같았다. 이제 알고리즘도 배우는데 이를 더 효율적으로 바꿀 방법이 있는가도 나중에 더 공부하고 찾아보면 좋을 것 같았다.