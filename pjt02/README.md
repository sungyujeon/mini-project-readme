# Project02 : <small>Python을 활용한 데이터 수집2</small>

###### 2021. 1. 29.(금)



<b>목표</b>

- Python 기본 문법 실습
- 파일 입출력에 대한 이해
- 데이터 구조에 대한 분석과 이해
- 데이터 가공 / JSON 형태 저장



## A. 영화 개수 카운트 기능 구현

>  To do
>
> 1. URLMaker 클래스 인스턴스 생성 후 `get_url()` 함수를 활용해 url 추출
>
> 2. url 주소를 request하여 반환된 response 객체를 `dict`로 변환
>
>  3. movie 리스트의 갯수를 count



##### | 완성 코드

```python
import requests
from tmdb import URLMaker

def popular_count():
    # URLMaker 클래스 인스턴스 생성
    maker = URLMaker('d35a0c8d750575819f0ef4311bf00c4f')

    # url
    url = maker.get_url()

    # response 객체 생성
    response = requests.get(url)

    # popular_movies_dict
    popular_movies_dict = response.json()

    # popular_movies_dict 
    cnt = len(popular_movies_dict.get('results'))

    return cnt


if __name__ == '__main__':
    print(popular_count())
```

<span style="color: #4285F4; font-size: 10pt">1. KMDB에서 발급받은 key를 init 함수의 인자로 넘겨 URLMaker 클래스의 인스턴스를 생성한다.</span>

<span style="color: #4285F4; font-size: 10pt">2. 기 정의한 URLMaker 클래스의 get_url()함수를 통해 인기 영화 목록을 가져와 dict 타입으로 변환한다.</span>

<span style="color: #4285F4; font-size: 10pt">3. 변환된 dict 타입의 영화 리스트의 갯수를 센다.</span>



## B. 특정 조건에 맞는 영화 출력

> To do
>
> 1. A에서 가져온 영화 리스트에서 평점 확인
> 2. 평점이 8점 이상인 영화 리스트만을 반환

| 완성 코드

```python
import requests
from tmdb import URLMaker
from pprint import pprint

def vote_average_movies():
    # URLMaker 클래스 인스턴스 생성
    maker = URLMaker('d35a0c8d750575819f0ef4311bf00c4f')

    # url
    url = maker.get_url()

    # response 객체 생성
    response = requests.get(url)

    # popular_movies_dict
    popular_movies_dict = response.json()

    # vote_average_movies_list
    vote_average_movies_list = []
    for movie in popular_movies_dict.get('results'):
        if movie.get('vote_average') >= 8:
            vote_average_movies_list.append(movie)
    
    return vote_average_movies_list


if __name__ == '__main__':
    pprint(vote_average_movies())
```

<span style="color: #4285F4; font-size: 10pt">1. A에서 얻은 영화 리스트를 for문을 돌면서 각 영화의 평점을 가져온다(`movie.get('vote_average').</span>

<span style="color: #4285F4; font-size: 10pt">2. 해당 평점이 8점 이상이면 vote_average_movies_list에 담아 반환한다.</span>



## C. 평점 순 정렬

> To do
>
> 1. A에서 얻은 영화 리스트를 평점 순으로 상위 

| 완성 코드

```python
import requests
from tmdb import URLMaker
from pprint import pprint

def ranking():
    # URLMaker 클래스 인스턴스 생성
    maker = URLMaker('d35a0c8d750575819f0ef4311bf00c4f')

    # url
    url = maker.get_url()

    # response 객체 생성
    response = requests.get(url)

    # popular_movies_dict
    popular_movies_dict = response.json()

    # popular_movies_list
    popular_movies_list = popular_movies_dict.get('results')

    # vote_average_top5_movies
    vote_average_top5_movies = sorted(popular_movies_list, key=lambda movie_dict: movie_dict.get('vote_average'))[-5:]

    # 내림차순 정렬
    vote_average_top5_movies.reverse()

    return vote_average_top5_movies
    

if __name__ == '__main__':
    # popular 영화 평점순 5개 출력
    pprint(ranking())
```

<span style="color: #4285F4; font-size: 10pt">1. A에서 가져온 영화 리스트를 `sorted()` 함수를 활용하여 정렬한다.</span>

<span style="color: #4285F4; font-size: 10pt">2. 이 때, 첫번째 인자로 정렬할 리스트를 넣고, 두번째 키워드 인자로 `key` 를 넣어서 정렬 기준이 되는 데이터를 넣는다.</span>

<span style="color: #4285F4; font-size: 10pt">3. 정렬 기준이 되는 데이터는 정렬할 list의 요소(dict)가 가지는 'vote_average' 평점이므로, lambda 함수를 사용하여 접근한다. 리스트의 개별 요소에서 가져오는 평점(movie_dict.get('vote_average')을 기준으로 정렬한다.</span>

<span style="color: #4285F4; font-size: 10pt">4. 정렬한 영화 리스트는 오름차순으로 정렬되어 있으므로 마지막 다섯개 영화 목록을 slicing을 통해 가져온 뒤, reverse()함수를 사용해 내림차순으로 재정렬하여 반환한다.</span>



## D. 제목 검색, 영화 추천

> To do
>
> 1. title을 기준으로 id값을 추출
> 2. id를 기준으로 추천영화 목록 조회
> 3. 추천 영화의 제목만을 추출하여 반환

| 완성 코드

```python
import requests
from tmdb import URLMaker
from pprint import pprint


def recommendation(title):
    # URLMaker 클래스 인스턴스 생성
    maker = URLMaker('d35a0c8d750575819f0ef4311bf00c4f')

    # title을 기준으로 id값 반환
    movie_id = maker.movie_id(title)

    # id값이 없는 영화이면 return None
    if not movie_id:
        return 
    
    # 추천영화 목록 조회 url 생성
    recommend_movie_url = maker.get_url('movie', f'{movie_id}/recommendations', region='KR', language='ko')

    # response 객체 생성
    response = requests.get(recommend_movie_url)

    # 추천 영화 리스트
    recommend_movie_list = response.json().get('results')

    # 추천 영화 제목 리스트
    recommend_movie_titles = []
    for recommend_movie in recommend_movie_list:
        recommend_movie_titles.append(recommend_movie.get('title'))
    
    return recommend_movie_titles


if __name__ == '__main__':
    # 제목 기반 영화 추천
    pprint(recommendation('기생충'))
    # =>   
    # ['원스 어폰 어 타임 인… 할리우드', '조조 래빗', '결혼 이야기', '나이브스 아웃', '1917', 
    # '조커', '아이리시맨', '미드소마', '라이트하우스', '그린 북', 
    # '언컷 젬스', '어스', '더 플랫폼', '블랙클랜스맨', '포드 V 페라리', 
    # '더 페이버릿: 여왕의 여자', '두 교황', '작은 아씨들', '테넷', '브레이킹 배드 무비: 엘 카미노']
    pprint(recommendation('그래비티'))    
    # => []
    pprint(recommendation('id없는 영화'))
    # => None
```

<span style="color: #4285F4; font-size: 10pt">1. URLMaker클래스의 `movie_id()` 함수를 이용하여 인자로 받은 title을 기준으로 id값을 추출한다.</span>

<span style="color: #4285F4; font-size: 10pt">2. A~C와는 다르게 추천 영화 목록을 받아야 하므로, 해당 영화의 id를 기준으로 get_url() 함수의 인자인 category, feature의 값을 movie/{movie_id}/recommendations로 변경하여 url을 추출한다.</span>

<span style="color: #4285F4; font-size: 10pt">3. 추천 영화 목록 리스트에서 영화 제목을 추출하여 반환한다.</span>



## E. 배우, 감독 리스트 출력

> To do
>
> 1. title을 기준으로 id값을 가져와 영화의 credits 정보를 추출
> 2. credits 정보에서 배우와 스태프 정보를 선별하여 반환

| 완성 코드

```python
import requests
from tmdb import URLMaker
from pprint import pprint


def credits(title):
    # URLMaker 클래스 인스턴스 생성
    maker = URLMaker('d35a0c8d750575819f0ef4311bf00c4f')

    # title을 기준으로 id값 반환
    movie_id = maker.movie_id(title)

    # id값이 없는 영화이면 return None
    if not movie_id:
        return 

    # credits url 생성
    credits_url = maker.get_url('movie', f'{movie_id}/credits')

    # response 객체 생성 및 json 변환
    response = requests.get(credits_url)
    credits_json = response.json()

    # cast & crew list
    credits_cast_list = response.json().get('cast')
    credits_crew_list = response.json().get('crew')

    # cast_id < 10인 배우
    casts = []
    for cast in credits_cast_list:
        if cast.get('cast_id') < 10:
            casts.append(cast.get('name'))

    # department가 Directing인 crew
    crews = []
    for crew in credits_crew_list:
        if crew.get('department') == 'Directing':
            crews.append(crew.get('name'))
    
    # credits dict
    credits_dict = {
        'cast': casts,
        'crew': crews
    }

    return credits_dict



if __name__ == '__main__':
    # id 기준 주연배우 감독 출력
    pprint(credits('기생충'))
    # => 
    # {
    #     'cast': [
    #         'Song Kang-ho',
    #         'Lee Sun-kyun',
    #         'Cho Yeo-jeong',
    #         'Choi Woo-shik',
    #         'Park So-dam',
    #         'Lee Jung-eun',
    #         'Chang Hyae-jin'
    #     ],
    #      'crew': [
    #         'Bong Joon-ho',
    #         'Han Jin-won',
    #         'Kim Seong-sik',
    #         'Lee Jung-hoon',
    #         'Park Hyun-cheol',
    #         'Yoon Young-woo'
    #     ]
    # } 
    pprint(credits('id없는 영화'))
    # => None
```

<span style="color: #4285F4; font-size: 10pt">1. D와 마찬가지로 title을 기준으로 id값을 추출한다. 이후 ㄱrecomendation이 아닌 credits를 인자로 넣어 credits 정보를 추출한다.</span>

 <span style="color: #4285F4; font-size: 10pt">2. credits 정보에서 cast와 crew 정보에 각각 접근하여 cast는 cast_id가 10 미만인 것만, crew는 department가 Directing인 정보만 가져와서 각 리스트를 dict에 담아 반환한다.</span>



## 마무리하며

1. pjt01에서는 이미 제공된 json 형태의 데이터를 가공하였다면, pjt02에서는 외부 API가 제공하는 정보를 가져와서 데이터를 가공하는 것을 배웠다. 이에 더하여 url을 생성하고 movie_id를 추출하는 등의 반복된 작업을 클래스 내 속성과 메서드로 작성하여 반복될 수 있는 코드를 줄였다. 실제 서비스하는 API를 가져오는 법과 API Document를 해석하고 활용하는 방법을 배울 수 있었다.
2. C번 문제에서 리스트 정렬을 다양하게 할 수 있을 듯한데 추가적인 학습이 필요할 듯하다. 리스트를 리스트 내 해당 요소로 정렬하는 것이 아니라 해당 요소들이 갖고 있는 정보를 기준으로 정렬한다면, sorted()와 lambda(익명 함수) 외에도 다양한 방법으로 구현할 수 있을 듯싶다. 페어 프로그래밍을 한 다른 동기는 영화 정보를 dictionary 형태로 가지고 있는 리스트에서 제목과 평점으로 구성되는 tuple을 만들었고, 이 튜플의 평점을 기준으로 정렬하여 영화 제목을 출력하는 방법을 사용했다. 이외에도 다양한 정렬 알고리즘을 활용해 문제를 해결할 수 있을 듯하다.