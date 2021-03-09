# 0408 webproject CRUD

1. models 만들때 IntegerField에 범위를 지정해주고 싶어서 검색해봤는데 max_length처럼 간단한 방법이 있는 줄 알았는데, 전혀 아니었다. 인터넷에 검색해 보니 여러가지 방법이 많았지만 그 중 가장 쉬워보이는 방법은 Validator를 쓰는 방법이었다.

```python
from django.db.models import IntegerField, Model
from django.core.validators import MaxValueValidator, MinValueValidator

class CoolModelBro(Model):
    limited_integer_field = IntegerField(
        default=1,
        validators=[
            MaxValueValidator(100),
            MinValueValidator(1)
        ]
     )
```

​	stack overflow에서 찾았는데 이러한 방향이 그나마 따라하기 쉬워보였다. 일단 다 해보고 models 지 	웟다가 시간 나면 해보기로 했다. 결국 시간이 부족해서 못했다. 아쉬움

2. admin 만들때 처음에 admin.py안에 적어줘야 하는 항목들 안 적고 아무생각없이 admin 만들었더니 게시글들을 하나도 안불러오는 문제가 발생했다. admin을 조금 수정해보고 다른 아이디 admin1으로 만들었는데도, 제대로 admin이 안되는걸 발견. 어떡해야 admin 을 제대로 등록 시킬수 있을까?

일단 그래도 성과는 admin 계정이 여러개 만들어 진다는 사실. 그리고 admin페이지에서 지우기도 손쉽게 가능하다. 나중에 알고보니 apps.py가 아니라 admin.py 을 건드렷어야 햇는데 둘이 비슷하게 생겨서 착각햇던거엿음.

3. review_list 가 처음에 review_llist.html에서 출력되지 않음.

/ community/templates/community 밑에 html들이 들어가야 하는데 community 는 만들어 놓고 그 안에 html이 좌르르 같은 디렉토리에 있으니까 이상한 점을 눈치 채지 못했었음. 그래서 디렉토리 변경했음. 그 뒤에 review_list for문으로 출력 시키려 했으나 나오지않음. urls에 문제가 있나 한참 찾았는데 review list에 h1태그를 입력해봐도 출력되지 않는걸 확인. 원인은 extends를 해놓고 block body를 안해서 발생한 문제. blockbody endblock으로 찾아줌. base.html extends를 해주면 항상 기본적으로 blockbody를 맨 위 아래에 넣어줘야 할 듯

2. html 수정중에 완료된 부분은 내버려두고, 틀린부분 참고 할 게 있어서 잠시 주석처리를 해두었는데, 주석처리에 조차 오류가 있으면 오류가 떠서 페이지가 출력이 안되는 걸 발견. html은 주석처리가 강력하지 않은 것 같다.

3. csrf token 계속 오타가 나서 endblock이 없다고 오류 메시지가 뜨길래 그게 문제인줄 알고 위치를 계속 바꿧다 .body를 씌웠다 지웠다 위치를 바꿨다 말았다 난리 쳤는데 알고 보니 csrf_token 이었다.. 언더바를 잊지 않도록 해야 할 것 같다.

4. 부트 스트랩 적용하는 과정에 navbar와 table은 나름대로 그 전에 해 본 경험이 있어서 이것저것 참고해서 만들어 보니 쉽게 만들 수 있었는데, form.html 이 너무 보노보노였다. 부트 스트랩 documentation 에서 form 예제를 찾아 다음과 같이 만들었다.

```python
<form>
  <div class="form-group">
    <label for="title"> Title </label>
    <input type="text" class="form-control" name="title" value="{{review.title}}">
    <small id="emailHelp" class="form-text text-muted">리뷰의 제목을 작성해주세요.</small>
  </div>
  <div class="form-group">
    <label for="movie_title"> Movie Title </label>
    <input type="text" class="form-control" name="movie_title" value="{{review.movie_title}}">
    <small id="emailHelp" class="form-text text-muted">영화의 제목을 작성해주세요.</small>
  </div>
  <div class="form-group">
    <label for="rank"> Rank </label>
    <input type="text" class="form-control" name="rank" value="{{review.rank}}">
    <small id="emailHelp" class="form-text text-muted">영화의 평점을 매겨주세요.</small>
  </div>
  <div class="form-group">
    <label for="exampleFormControlTextarea1"> 내 용 </label>
    <input type="text" class="form-control" name="content" value="{{review.content}}">
    <small id="emailHelp" class="form-text text-muted">리뷰를 작성해 주세요.</small>
  </div>
  </div>
  <button type="submit" class="btn btn-primary">Submit</button>
  <a href="{% url 'community:review_list' %}"> Back </a>
</form>
```

이렇게 까지 열심히 했는데 이 방법으론 도저히 input값이 넘어가지가 않았다. 문제점을 잘 모르겠다..

그래서 form을 좀 더 쉽게 받을 수 있는 방법이 없나 하고 찾아보니 bootstrap4에 crispy forms 라는 앱이 있다는 것을 알게 되었다. 바로 pip install django-crispy-forms 깔고, installed apps 에 crispy-forms를 등록해주고, 제공해주는 템플릿으로 만드니까 form 도 보노보노를 벗어나게 되었다.

```
{% extends 'base.html' %}

{% load crispy_forms_tags %}

{% block body %}
<div class="d-flex justify-content-center">
  <form method="post" novalidate>
    {% csrf_token %}
    {{ form|crispy }}
    <button type="submit" class="btn btn-success">Save</button>
  </form>
</div>
{% endblock %}
```

그런데 기본적으로는 왼쪽 정렬되어 있어서 너무 보기 안 좋길래, bootstrap에서 form 태그를 div로 묶어 flex로 만들어 중앙으로 정렬해줬다.

그리고 이유는 모르겠지만 예제에는 block content로 제공 되었는데 block content는 출력 되지 않았고 block body로 바꾸니까 출력이 되었다. 왜인지는 모르겠음.

1. 중간 중간 url을 부트스트랩에 적용시키는 과정에서 하드코딩으로 넣은 url들이 제대로 url 위치를 적용 받지 못하는 문제점이 많이 발생했었다. {% url '이름' Review.pk %} 형태를 잊지 않아야 할 것 같다.
2. html에서 에러가 난다고 나오지만 실제로는 views에서 에러가 나서 문제가 되는 곳이 종종 있었다. Error 메시지의 가장 커다란 부분만 읽지 말고 세밀하고 꼼꼼하게 읽어야 할 것 같다.
3. 특히 html에서 뜬금없이 bootstrap CDN에서 오류가 뜬다고 하는 문제가 많이 발생했는데, 전혀 틀린 부분이 없었다. 알고보니 url설정이 잘 못 되어 있었는데 그곳의 {} 가 다른 위치에서 오류를 발생시키기도 하는 것 같았다.
4. 부트스트랩 테이블에 Review_list들을 출력 시키는데 크기를 내 맘대로 하고 싶었으나, 잘 되지 않았다. class 에 col-숫자 총 합이 12가 되도록 설정하는 걸로 기억했는데, 의도치 않게 우측 정렬되거나 비율이 맞지 않았다.

