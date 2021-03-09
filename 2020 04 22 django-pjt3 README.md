## 2020 04 22 django-pjt3 README

#### 1. 역할분배

팀 역할 배분에 관해 고민을 그 전에 많이 했었는데, 그냥 서로 다른 기능을 만들다가 괜히 이상한 부분에서 오류가 나서 못 잡아낼 바에야, 그냥 계속 전화통화를 하면서 한 화면을 보는 것이 오류를 가장 적게 만들어 낼 것 같아서, 7:3보다는 5:5의 비율로 서로 코딩을 완성해 나가기로 하였다.

#### 2. 구현순서

명세서를 읽고, 기능을 만들 계획을 navbar / accounts / community 순서로 만들기로 했다. base.html 인 navbar가 먼저 있어야 다른 html들간의 페이지 이동이 쉬울 것이라 생각했기 때문이다. 후에 navbar는 먼저 만든 건 잘한 일이었지만, accounts보다 community를 먼저 만들걸 하고 후회했다.

#### 3. 기본구성

settings, base.html, 부트스트랩 설정, 기본적인 navbar 구성부터 했다. 초반은 그래도 몇번 해본 과정이 대부분이라 그리 어렵지 않았다. base_dir을 지정해주는 것은 까다로우니 조심해야 한다.

#### 4. Accounts

##### 1) 로그인 기능 http method POST를 처리하는 방법 중 명세서에 표현이 어려운 부분이 있었다.

```
2. 로그인 하기 전, 기존 URL이 함께 넘어왔다면 해당 URL로,
3. 넘어오지 않았다면 전체 리뷰 목록 페이지로 redirect 합니다. 데이터가 유효하지 않다면
	에러 메시지를 포함하여, 데이터를 작성하는 form 을 사용자 화면에 표시합니다. 
```

 여기에서 처음에 '''기존 url이 함께 넘어왔다면''' 이라는 문장이 어떤 말인지를 이해하기가 힘들었다. 그러나 이내 이게 auth_login 을 할 때, detail페이지이든 , review_list 페이지 이든 여러 url에서 login 을 들어갔을 경우를 상정한 문장이라는 것으로 이해하고, 크게 무리 없이 넘어 갈 수 있었다.

##### 2) 하드타이핑

 처음엔 render와 redirect 모두 URL을 하드타이핑 하지 않고, 이름을 읽는 방식으로 작성해서 서버를 열어 보았으나, 에러가 발생함을 알아채고 render 부분은 하드타이핑 방식으로 구현하여 에러를 제거하였다.

##### 3) 모델링의 중요성

account 기능은 기본 django가 사용하는 User Creation Form을 사용했기에 , 모델링을 하지 않고 진행을 했다. 차후 community기능을 구현 한 뒤에 Form을 만들고 그제서야 migration과 admin 기능을 사용 할 수 있었는데, 그렇다보니까 accounts를 작업할 때, 서버에서 하나하나 지금껏 작성한 코드가 맞는지 확인 할 수가 없어서 어려움을 겪었다. 작업의 순서를 설정할 때, 기본 crud 기능을 먼저 구현해두어야 admin superuser 기능을 사용할 수 있기 때문에, migrations를 먼저 해두어야 작업이 용이하다는 것을 깨달았다.

##### 4) is authenticated

```html
{% if user.is_authenticated %}
{% else %}
{% endif %}
```

와 같은 방식으로 유저가 권한이 있을 때 html에서 표시되는 부분, 표시 되지 않는 부분을 따로 선정해서 출력되게 만들었다.

##### 5) form action의 url 설정

#### 5. community

##### 1) CRUD 기능 순서별로 구현

##### 2) @login_required

```python
from django.contrib.auth.decorators import login_required
```

login required라는 decorators를 어느곳에서 불러와야 하는지 어려움을 겪었는데, 탁희 강사님의 수업 노트를 보고 찾았다. 그 뒤 login이 필요한 기능들 위에 모두 적용 시켜 주었다.

##### 3) detail , update, delete에서 review_pk. pk=review_pk

review_pk , pk=pk, pk=review_pk 등등.. pk를 적용 시키는 부분에서 많이 헷갈려 오류가 많이 발생했었다. 서버를 열어볼 때마다 새로운 오류가 새로운 곳에서 발생하는 것이 신기했다.

##### 4) redirect페이지를 어디로 연결해줘야 하는지 세세하게 따져봐야함

명세서에서 어떠한 경우 어느 페이지로 연결에 관해 굉장히 세세하게 명세되어 있었는데, 그냥 다 review_list로 연결해버리고 싶은 충동을 느꼈다. 인내심을 가지고 진행해서 명세표 대로 원하는 곳으로 redirect될 수 있도록 등록 하였다.

#### 6. admin 등록

```python
from django.contrib import admin
from .models import Review, Comment
# Register your models here.
admin.site.register(Review)
admin.site.register(Comment)
```

 admin을 위해 review와 comment를 둘 다 불러와 줘야 하는 것을 잊지 않기.

#### 7.가장 큰 어려움 models.py 와 forms.py

```python
from django import forms
from .models import Review, Comment

class ReviewForm(forms.ModelForm):
    class Meta:
        model = Review
        fields = ['title','movie_title','rank','content']

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content',]
from django.db import models
from django.conf import settings

# Create your models here.
class Review(models.Model):
    title = models.CharField(max_length=100)
    movie_title = models.CharField(max_length=30)
    rank = models.IntegerField()
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    user = models.ForeignKey(settings.AUTH_USER_MODEL,on_delete=models.CASCADE)

class Comment(models.Model):
    content= models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    review = models.ForeignKey(Review, on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL,on_delete=models.CASCADE)
```

Community 기능을 구현할 때, model 관련한 명세서에서 user를 등록해줘야 한다고 했었다. review와 comment에는 어짜피 각 글의 pk값이 알아서 저장될 것이라고 생각해서 처음에 만들때

```python
user = models.ForeignKey(settings.AUTH_USER_MODEL,on_delete=models.CASCADE)
```

이 부분에 관한 생각을 해주지 못한 채, migrations를 진행 했었다.

나중에 각 review 별로 그것을 저장한 user의 Foreign key 값이 필요하다는 것을 깨닫고, migrations를 다시 해야 하는 어려움에 처했었다.

그래서 migrations 내에 "**init**.py" 파일은 내버려두고, 나머지 파일을 지우고, dblite3도 지우고 다시 세팅을 했어야 했는데, 이 부분에서 init 파일까지 통채로 지워버려서 지금까지 만들어놓은 더미데이터도 날리고, 기능들도 정상 작동하지 않는 부분이 있어 1시간 가량을 낭비 했었다.

처음에 model을 만들 때 더욱 조심했어야 하는 부분이라고 생각한다. 특히 ForeignKey에서 settings를 지정해줬어야 하는 부분이 미숙해서 더욱 어려웠던 것 같다.

form도 fields를 all로 지정해줬더니, user의 foreign key가 이상하게 출력되어 보여지는 경우가 발생해서, 결국 user를 제외한 모든 부분을 하나하나 입력해줘야 한다.

#### 8. html 파일들에서 request.user == review.user 가 같을때에만 update와 delete가 출력하게 하는 기능

```python
{% if request.user == review.user %}
<a href="{% url 'community:update' review.pk %}">UPDATE</a>
<a href="{% url 'community:delete' review.pk %}">DELETE</a>
{% endif %}
```

이런식으로 작성하여서, 로그인한 user와 리뷰를 쓴 user가 같을 때에만 수정/삭제가 가능하도록 만들었다.

#### 최종 느낀 점:

팀프로젝트에서 각 기능별로 수행을 하거나, 7:3으로 한명이 치고, 한명이 가이드 하는 방향 보다 2명이 같이 서로서로 동시에 한 화면을 작성하는 5:5방식으로 했는데, 크게 속도가 뒤쳐지지도 않고, 특히 오타가 많은 ***의 실수를 그때그때 잡을 수 있어서, 우리팀에겐 이 방식이 제일 나은 팀워크 방식이었다고 생각한다.

마지막에 서버를 켜보면, 주황색은 ***가 쓴 부분이고, 분홍색은 내가 쓴 부분이었는데 두가지 색이 얼기설기 얽혀 있는 걸 보며 뿌듯함을 많이 느꼈다.

구현해야 할 기능들이 생각보다 디테일하게 많아서 시간이 오래걸렸는데, 아직도 auth 부분이나, foreignkey 를 통해 권한 별로 기능을 구현하는 디테일까지는 완벽하게 구현하지 못해서 못내 아쉬움이 남는다.

하지만 그래도 거의 대부분의 기능을 구현했고, 오류 없이 돌아가는 페이지를 만들었고 + 팀원과 싸우지 않을 수 있음에 감사했다. 마지막으로 막판에 살짝 msg를 뿌려준 *** 형에게도 Big shout out!