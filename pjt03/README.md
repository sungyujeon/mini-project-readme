#  PJT03_반응형 웹 페이지 구성



>  목표
>
> - HTML을 통한 웹 페이지 마크업 분석
> - CSS 라이브러리의 이해와 활용
> - 컴포넌트 및 그리드 시스템 활용
> - 커뮤니티 서비스 반응형 레이아웃 구성



이번 반응형 웹페이지 구성 프로젝트에서는 'HTML, CSS 문법', 'Bootstrap Components', 'Bootstrap Grid System'의 이해가 필요했다. HTML, CSS는 MDN 문서를 참고하였고, Bootstrap은 공식 홈페이지의 Documentation을 참고했다. 

전체 HTML의 구조(뼈대)와 각 요소들에 적용되는 스타일은 기본적으로 Bootstrap에서 제공하는  API를 활용하였다.



> 어려웠던 내용

## Navbar

네비게이션 바는 웹페이지 상단의 메뉴들을 보여주는 component로, 웹페이지의 로고와 구조적인 메뉴들의 제목 정보를 담는다. 사용자들은 네비게이션 바를 통해 특정 페이지에 접근하므로 상단에 고정하여 지속적으로 보여줄 필요가 있었다. 이 때, Home, Community, Login 메뉴들이 오른쪽으로 정렬되지 않는 경우가 생겼는데 아래와 같이 해결하였다.

```css
@media (min-width: 768px) {
  .navbar-collapse {
    width: auto;
  }
}
```

##### `class="navbar-collapse"`

semantic 태그를 활용해 navbar라는 것을 명시적으로 선언한다. 부트스트랩에서 제공하는 navbar 클래스는  `display: flex; justify-content: space-between` 속성이 포함된다. navbar에 속하는 요소들을 flex item으로 만들어 좌우로 벌어지게 정렬한다. 이 때, navbar-collapse 클래스는 `width: 100%`로 되어 있는데, 이 속성은 inline-block의 나머지 width를 모두 차지하게 만들어 justify-content-between 속성이 적용되지 않게 했다. 따라서 Home, Community, Login 메뉴들이 오른쪽으로 가게 하기 위해 아래와 같이 media query를 만들어 width가 768px을 넘어가면 width를 auto로 만들어  justify-content-between 스타일이 적용되게 만들었다.



## Card image

card의 image를 넣으면 image의 원본 크기에 따라 카드의 크기도 함께 조정이 되었다. 카드의 가로 길이는 고정이 되어있지만, 이미지의 크기에 따라(원본 이미지의 세로 길이가 충분히 길다면) 카드의 이미지도 그에 따라 함께 늘어나는 현상이 발생하였다. 

이를 해결하기 위해서 같은 조원은 카드의 width와 height를 명시적으로 고정시키는 방법을 선택했다. 하지만 이 방법도 원본 이미지가 가로로 길다면 이미지가 너무 작아지는 현상이 발생할 수 있다. 

따라서 두 가지 방법을 선택할 수 있다. 

1)카드의 가로 세로 길이를 고정하고 원본 이미지의 비율과 크기를 비슷하게 맞추는 방법 2) 카드의 크기를 고정하지 않고 원본 이미지에 따라 유동적으로 카드의 크기를 조정하는 방법. 두가지를 상황에 맞게 사용하는 것이 필요하다고 느꼈다.

찾아보니 핀터레스트 방식의 웹사이트에서 이미지 비율에 따라서 출력을 해주는 것을 보았는데, 테트리스 쌓듯이 원본 비율을 유지한 채로 일정한 규칙에 따라 이미지들을 보여주는 것이다. 위의 2)번 방법을 사용할 때 적용해볼 수 있을 듯하다.



## 반응형 웹에서 display: none

특정 px의 웹페이지에서 전혀 다른 레이아웃으로 컨텐츠를 보여주는 방식이다. 영화 정보를 디바이스 크기에 따라 보여주는 방식을 다르게 해주는 것인데, 처음에는 table 구조를 유지한 채로 구현을 해보려고 했으나 불가능하였다. 따라서 서로 다른 div 태그에서 같은 영화 정보를 담아 필요에 따라 보여주는 컨텐츠를 취사 선택하게 하였다. 

```html
<html>
  <!-- contents1 -->
  <div class="d-none d-lg-block col-lg-10">
    
  <!-- contents2 -->
  <div class="col-12 d-block d-lg-none">
</html>
```

위와 같이 필요한 컨텐츠를 디바이스 크기에 따라 별도로 출력하는 코드다. 이 때 주의해야 할 점이 display none인 상태에서 block으로, 또는 block에서  none으로 바꾸어야 한다는 것이다. min-width, max-width에 따라 특정 범위에서 스타일이 적용되기 때문에, 명시적으로 구분을하여 코드를 짜야했다.



## 마치며

1주일 간 HTML, CSS, Bootstrap을 진행하다 보니 헷갈리는 부분이 많았다. 양이 너무 방대하다보니 필요에 따라 documentation을 참고해서 적용시키는 방식이 더 효율적인 듯싶다. 이외에도 animation과 font 등 API를 필요에 따라 추가시켜서 사용해야 하는 경우도 많다. 꾸준한 학습이 필요해 보인다.