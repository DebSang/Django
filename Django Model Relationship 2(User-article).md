## Model Relationship 2

> 1 : N 관계 설정

- User - Article



> User 모델 참조하기

1.  settings.AUTH_USER_MODEL
   - User 모델에 대한 외래 키 또는 다대다 관계를 정의 할 때 사용해야 함
   - <u>models.py에서 User 모델을 참조</u> 할 때 사용

2. get_user_model()
   - 현재 활성화(active)된 User 모델을 반환
     - 커스터마이징한 User 모델이 있을 경우는 Custom User 모델, 그렇지 않으면 User를 반환
     - User를 직접 참조하지 않는 이유!!!!
   - <u>models.py가 아닌 다른 모든 곳에서</u> User 모델을 참조할 때 사용



>User와 Article 간 모델 관계 정의 후 migration

```python
#articles/models.py
from django.conf import settings

class Article(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
```

- 모델 관계를 정의 후 makemigrations를 하면 null값이 허용되지 않는 user_id 필드가 별도의 값 없이 article에 추가되려 하기 때문에 무언가를 입력하라는 창이 뜸.
- 첫 입력창에 1을 입력후 enter, '현재 화면에서 기본 값을 설정하겠다' 라는 의미
- 두번째 입력창에 역시 1을 입력 후 enter, '기존 테이블에 추가되는 user_id 필드의 값을 1로 설정하겠다'라는 의미.
- migrate로 마무리



- 게시글 작성 페이지에서 불필요한 User 필드가 출력되는 것을 확인
  - forms.py에서 user 필드 제외 시켜줌

```python
#articles/forms.py
class ArticleForm(forms.ModelForm):

    class Meta:
        model = Article
        exclude = ('user',)
```



- 이후 게시글 작성 시 NOT NULL constraint failed: articles_article.user_id 에러 발생
  - 게시글 작성 시 작성자 정보(article.user)가 누락 되었기 때문

> CREATE

```python
@login_required
@require_http_methods(['GET', 'POST'])
def create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save(commit=False)
            #article 안에 user의 정보가 누락되었기 때문에 form.save(commit=False)로 저장은 안하고 데이터만 받아와서 article.user 값을 준다.
            article.user = request.user
            article.save()
            return redirect('articles:detail', article.pk)
```



> CREATE

- 자신이 작성한 게시글만 삭제 가능하도록 설정

```python
def delete(request, pk):
    article = get_object_or_404(Article, pk=pk)
    if request.user.is_authenticated:
        if request.user == article.user:
            article.delete()
            return redirect('articles:index')   
    return redirect('articles:index', article.pk)
```



> UPDATE

- 자신이 작성한 게시글만 수정 가능하도록 설정

```python
@login_required
@require_http_methods(['GET', 'POST'])
def update(request, pk):
    article = get_object_or_404(Article, pk=pk)
    if request.user == article.user:
        if request.method == 'POST':
            form = ArticleForm(request.POST, instance=article)
            if form.is_valid():
                article = form.save()
                return redirect('articles:detail', article.pk)
        else:
            form = ArticleForm(instance=article)
    else: 
        redirect('articles:index')
    context = {
        'article': article,
        'form': form,
    }
    return render(request, 'articles/update.html', context)
```



> READ

- 게시글 작성 user가 누구인지 index.html에서 출력하기

```html
{% extends 'base.html' %}

{% block content %}
	...
  <hr>
  {% for article in articles %}
    <p>작성자 : {{ article.user }}</p>
    <p>글 번호: {{ article.pk }}</p>  
    <p>글 제목: {{ article.title }}</p>
    <p>글 내용: {{ article.content }}</p>
    <a href="{% url 'articles:detail' article.pk %}">DETAIL</a>
    <hr>
  {% endfor %}
{% endblock content %}
```

- 해당 게시글의 작성자가 아니라면, 수정/삭제 버튼을 출력하지 않도록 처리

```html
{% if request.user == article.user %}
  <a href="{% url 'articles:update' article.pk %}">수정</a>
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="삭제">
  </form>
{% endif %}
```