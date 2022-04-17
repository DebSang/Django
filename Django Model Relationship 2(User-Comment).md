## Model Relationship 2

> 1 : N 관계 설정

- User - Commnet



> User와 Comment 간 모델 관계 정의 후 migration

```python
class Comment(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    ...
```

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```

- 모델 관계를 정의 후 makemigrations를 하면 null값이 허용되지 않는 user_id 필드가 별도의 값 없이 article에 추가되려 하기 때문에 무언가를 입력하라는 창이 뜸.
- 첫 입력창에 1을 입력후 enter, '현재 화면에서 기본 값을 설정하겠다' 라는 의미
- 두번째 입력창에 역시 1을 입력 후 enter, '기존 테이블에 추가되는 user_id 필드의 값을 1로 설정하겠다'라는 의미.
- migrate로 마무리



> 댓글 출력 필드 수정

- 게시글 작성 페이지에서 불필요한 User 필드가 출력되는 것을 확인
- content만 출력되도록 설정

```python
class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        fields = ('content',)
```

- 이후 게시글 작성 시 NOT NULL constraint failed: articles_article.user_id 에러 발생

- 게시글 작성 시 작성자 정보(article.user)가 누락 되었기 때문



> CREATE

```python
def comment_create(request, pk):
    if request.user.is_authenticated:
        article = get_object_or_404(Article, pk=pk)
        comment_form = CommentForm(request.POST)
        if comment_form.is_valid():
            comment = comment_form.save(commit=False)
            comment.article = article
            comment.user = request.user
            comment.save()
        return redirect('articles:detail', article.pk)
    return redirect('accounts:login')
```



> READ

- 비로그인 유저에게는 댓글 form 출력 숨기기

```python
{% if request.user.is_authenticated %}
    <form action="{% url 'articles:comment_create' article.pk %}" method="POST">
      {% csrf_token %}
      {{ comment_form }}
      <input type="submit">
    </form>
{% else %}
    <a href="{% url 'accounts:login' %}">댓글을 작성하려면 로그인 하세요</a>
{% endif %}
```

- 댓글 작성자 출력하기

```html
{% for comment in comments %}
      <li>
        {{ comment.user }} : {{ comment.content }}
        {% if request.user == comment.user %}
          <form action="{% url 'articles:comment_delete' article.pk comment.pk %}" method="POST">
            {% csrf_token %}
            <input type="submit" value="삭제">
          </form>
        {% endif %}
      </li>
{% empty %}
	<p> 댓글이 없어요. </p>
{% endfor %}
```

