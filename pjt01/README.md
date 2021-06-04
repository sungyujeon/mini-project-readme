# Project01 : <small>Python을 활용한 데이터 수집</small>

###### 2021. 1. 22.(금)



<b>목표</b>

- Python 기본 문법 실습
- 파일 입출력에 대한 이해
- 데이터 구조에 대한 분석과 이해
- 데이터 가공 / JSON 형태 저장

<span style="color: #4285F4; font-size: 10pt">첫 프로젝트에서는 Python을 활용한 데이터 수집에 대해 진행한다. 이를 통해 기본 문법을 익히고, 파일 입출력에 대해 이해한다. 데이터 구조를 분석하고 이해할 수 있다. 데이터를 가공하여 JSON 형태로 저장할 수 있다.</span>



## A. 제공되는 영화 데이터의 주요내용 수집

>  To do
>
> 1. 샘플데이터인 `movie.json` 파일과 데이터를 load
>
> 2. 위 데이터 중 다음을 key 값으로 갖는 value만 추출
>
>    `id, title, poster_path, vote_average, overview, genre_ids`
>
> 3. 위 정보를 dictionary로 반환하는 함수 movie_info 정의



##### | 완성 코드

```python
import json
from pprint import pprint


def movie_info(movie):
    # 명세에 정의된 key 데이터 추출
    mId = movie.get('id')
    title = movie.get('title')
    poster_path = movie.get('poster_path')
    vote_average = movie.get('vote_average')
    overview = movie.get('overview')
    genre_ids = movie.get('genre_ids')
    
    # key 데이터를 dictionary 형태로 저장
    movie_dict = {
        'id': mId,
        'title': title,
        'poster_path': poster_path,
        'vote_average': vote_average,
        'overview': overview,
        'genre_ids': genre_ids,
    }
    return movie_dict

# 아래의 코드는 수정하지 않습니다.
if __name__ == '__main__':
    movie_json = open('data/movie.json', encoding='UTF8')
    movie_dict = json.load(movie_json)
    pprint(movie_info(movie_dict))
```

<span style="color: #4285F4; font-size: 10pt">1. 첫번째 요구사항인 샘플데이터 파일을 가져와 JSON 형태로 load하는 것은 이미 정의되어 있다.</span>

<span style="color: #4285F4; font-size: 10pt">2. 명세에서 요구한 데이터는 `movie_info()` 함수에서 JSON 형태의 데이터인 `movie`에서 `get()` 함수를 통해 추출한다.</span>

<span style="color: #4285F4; font-size: 10pt">3. 위 과정에서 추출한 데이터를 `movie_dict`에 dictionary 형태로 담아 `return`한다.</span>



## B. 제공되는 영화 데이터의 주요내용 수정

> To do
>
> 1. A에서 genre_ids를 `genres.json` 파일을 활용해 genre_names로 변환
> 2. genre_names로 대체된 dictionary를 반환하는 `movie_info` 함수 완성

| 완성 코드

```python
import json
from pprint import pprint


def movie_info(movie, genres):
    # 명세에 정의된 key 데이터 추출
    mId = movie.get('id')
    title = movie.get('title')
    poster_path = movie.get('poster_path')
    vote_average = movie.get('vote_average')
    overview = movie.get('overview')

    # genres에서 movie_dict의 genre_ids에 대응되는 name 추출
    genre_names = []
    for genre_id in movie.get('genre_ids'):
        for id_dict in genres:
            if genre_id == id_dict.get('id'):
                genre_names.append(id_dict.get('name'))

    # key 데이터를 dictionary 형태로 저장
    movie_dict = {
        'id': mId,
        'title': title,
        'poster_path': poster_path,
        'vote_average': vote_average,
        'overview': overview,
        'genre_names': genre_names,
    }
    
    return movie_dict  
        

# 아래의 코드는 수정하지 않습니다.
if __name__ == '__main__':
    movie_json = open('data/movie.json', encoding='UTF8')
    movie = json.load(movie_json)

    genres_json = open('data/genres.json', encoding='UTF8')
    genres_list = json.load(genres_json)

    pprint(movie_info(movie, genres_list))
```

<span style="color: #4285F4; font-size: 10pt">1. JSON 데이터인 `movie`에서 `genre_ids`를, `genres`에서 id와 name을 key로 갖는 `id_dict`를 가져와 이중 for문을 통해 `genre_id`와 `id_dict의 id`값이 일치할 때 해당 genre의 name을 `genre_names` 리스트에 append</span>

<span style="color: #4285F4; font-size: 10pt">2. `genre_names` 리스트를 `genre_ids` 대신에 `movie_dict`에 추가</span>



## C. 다중 데이터 분석 및 수정

> To do
>
> 1. 20개의 영화데이터에서 B까지의 데이터를 추출한 영화 리스트 반환

| 완성 코드

```python
import json
from pprint import pprint


def movie_info(movies, genres):
    # 명세에 정의된 key 데이터만을 포함한 movies_list 초기화
    movies_list = []

    for movie in movies:

        # 명세에 정의된 key 데이터 추출
        mId = movie.get('id')
        title = movie.get('title')
        poster_path = movie.get('poster_path')
        vote_average = movie.get('vote_average')
        overview = movie.get('overview')

        # genres에서 movie_dict의 genre_ids에 대응되는 name 추출
        genre_names = []
        for genre_id in movie.get('genre_ids'):
            for id_dict in genres:
                if genre_id == id_dict.get('id'):
                    genre_names.append(id_dict.get('name'))

        # key 데이터를 dictionary 형태로 저장
        movie_dict = {
            'id': mId,
            'title': title,
            'poster_path': poster_path,
            'vote_average': vote_average,
            'overview': overview,
            'genre_names': genre_names,
        }

        # movie_dict를 movies_list에 추가
        movies_list.append(movie_dict)
    
    return movies_list
        
        
# 아래의 코드는 수정하지 않습니다.
if __name__ == '__main__':
    movies_json = open('data/movies.json', encoding='UTF8')
    movies_list = json.load(movies_json)

    genres_json = open('data/genres.json', encoding='UTF8')
    genres_list = json.load(genres_json)

    pprint(movie_info(movies_list, genres_list))
```

<span style="color: #4285F4; font-size: 10pt">1. JSON 형태의 데이터인 `movies`를 반복문을 통해 개별 movie에 대한 B의 데이터 추출 과정을 반복</span>

<span style="color: #4285F4; font-size: 10pt">2. 개별 movie_info를 담는 movies_list에 추가</span>



## D. 알고리즘을 통한 데이터 출력

> To do
>
> 1. 개별 movie의 정보를 담고 있는 JSON 파일을 load
> 2. JSON 형태의 데이터에서 `revenue` 정보를 추출
> 3. 반복문을 통해 max_revenue 리턴

| 완성 코드

```python
import json


def max_revenue(movies):
    # max_revenue와 max_revenue를 갖는 영화제목 초기화
    max_revenue = 0
    max_revenue_title = ''

    # 각각의 movie에 대한 max_revenue 비교
    # 현재까지의 max_revenue와 새롭게 가져온 데이터의 revenue를 비교하여 큰 값을 할당
    for movie in movies:
        mId = movie.get('id')
        movie_json = open(f'data/movies/{mId}.json', encoding='UTF8')
        movie_info = json.load(movie_json)

        revenue = movie_info.get('revenue')
        
        if max_revenue < revenue:
            max_revenue = revenue
            max_revenue_title = movie_info.get('title')

    return max_revenue_title
        
 
if __name__ == '__main__':
    movies_json = open('data/movies.json', encoding='UTF8')
    movies_list = json.load(movies_json)
    
    print(max_revenue(movies_list))
```

<span style="color: #4285F4; font-size: 10pt">1. C의 과정에서, 개별 movie에 대한 `revenue`정보를 가져와 이전 영화까지의 `max_revenue`와 대소비교를 통해 더 큰 값을 할당하여 리턴</span>



## E. 알고리즘을 통한 데이터 출력

> To do
>
> 1. 개별 영화 정보에서 개봉일 정보(release_date)를 추출
> 2. 개봉일 정보에서 12월 개봉일 정보를 가진 영화를 선별 추출하여 해당 영화 제목들을 출력

| 완성 코드

```python
import json
from datetime import date

def dec_movies(movies):
    dec_movies = []

    for movie in movies:
        mId = movie.get('id')
        movie_json = open(f'data/movies/{mId}.json', encoding='UTF8')
        movie_info = json.load(movie_json)

        # release_date
        release_date = movie_info.get('release_date')
        
        # string slicing으로 release_date month 추출
        #dec_release_date = release_date[5:7]
        
        # datetime 모듈로 추출
        release_month = date.fromisoformat(release_date).month
```

<span style="color: #4285F4; font-size: 10pt">1.`string slicing`:  release_date에서 str 타입의 날짜에서 `month` 정보만 따로 추출하기 위해서 처음에 string slicing 방법을 사용했다. 날짜 형식이 `'YYYY-MM-DD'` 형식으로 고정되어 있기 때문에, [5:7] slicing으로 월 정보를 추출하였다.</span>

 <span style="color: #4285F4; font-size: 10pt">2. `fromisoforamt`: 함께 코드를 리뷰한 동기는 date 클래스의 `fromisoformat()` 함수를 사용하여 month 정보를 추출하였다. `datetime` 내장 모듈의 `date` 클래스는 날짜를 표현하는 데 사용되고, `fromisoformat()` 함수는 `YYYY-MM-DD` 형태의 문자열을  `date` 객체로 반환해준다. 이 때, year/month/day 인스턴스에 접근하여 년/월/일 정보를 추출할 수 있다. 따라서 `date.fromisoformat(release_date).moth`로 month 정보를 추출할 수 있다는 것을 배웠다.</span>



```python
        if release_month == 12:
            dec_movies.append(movie_info.get('title'))

    return dec_movies
        

# 아래의 코드는 수정하지 않습니다.
if __name__ == '__main__':
    movies_json = open('data/movies.json', encoding='UTF8')
    movies_list = json.load(movies_json)
    
    print(dec_movies(movies_list))
```

<span style="color: #4285F4; font-size: 10pt">1. C에서 추출한 데이터에서 release_date 데이터를 별도 추출</span>

<span style="color: #4285F4; font-size: 10pt">2. release_date를 datetime의 fromisoformat() 함수를 이용하여 추출</span>

<span style="color: #4285F4; font-size: 10pt">3. 12월에 개봉한 영화의 제목을 리스트에 추가하여 반환</span>



