##### 로그아웃

> 로그아웃은 session을 Delete하는 로직과 같음

- logout 함수
  - logout(request)
    - 사용자가 로그인 하지 않은 경우에도 오류 발생 X
    - 현재 요청에 대한 session data를 DB에서 완전히 삭제, 클라이언트의 쿠케이서도 sessionid가 삭제됨
    - views의 logout 함수와 구별하기 위해 기존의 logout 함수를 auth_logout으로 이름 변경

>accounts/urls.py

```python
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    path('logout/', views.logout, name='logout'),
    ]
```

>accounts/views.py

```python
from django.contrib.auth import logout as auth_logout 
from django.views.decorators.http import require_POST

@require_POST
def logout(request):
    if request.user.is_authenticated:
        auth_logout(request)
    return redirect('articles:index')
```

>로그아웃 표시가 로그인 상태에서만 보이도록 base.html에 작성

```html
<body>
	<div class="container">
        {% if request.user.is_authenticated %}
            <h3>Hello, {{ user }}</h3>
        	<form action="{% url 'accounts:logout' %}" method="POST">
        	{% csrf_token %}
        	<input type="submit" value="Logout">
      		</form>
        {% else %}
        	<a href="{%url 'accounts:login'%}">Login</a>
        {% endif %}
        {% block content %}
        {% endblock content %}
    </div>
</body>
```



##### 로그인 사용자에 대한 엑세스 제한 2가지 방법

> 로그인과 비로그인 상태에서 다르게 작동하도록 설정

1. The raw way

   - is_authenticated attribute

   > articles/index.html 에도 적용!!!

   ```jinja2
   {% extends 'base.html' %}
   
   {% block content %}
     <h1>Articles</h1>
     {% if request.user.is_authenticated %}
       <a href="{% url 'articles:create' %}">CREATE</a>
     {% else %}
       <a href="{% url 'accounts:login' %}">[새 글을 작성하려면 로그인 하세요]</a>
     {% endif %}
     <hr>
   	...
   {% endblock content %}
   ```

   

2. The login_required decorator

   - 사용자가 로그인되어 있지 않으면, settings.LOGIN_URL에 설정된 문자열 기반 절대경로로 redirect 함

     - 기본값은 '/accounts/login/', app이름을 accounts로 했던 이유 중 하나

   - 인증 성공 시 사용자가 redirect 되어야 하는 경로로 "netx"라는 쿼리 문자열 매개 변수에 저장됨

     - 예시) /accounts/login/?next=/articles/create/

     - 현재 URL(next parameter가 있는) 요청을 보내기 위해 login.html의 form태그에서 action값을 비우기

     - views.py의 login 함수에서 리턴값을 다음과 같이 설정

       ```python
       return redirect(request.GET.get('next') or 'articles:index')
       ```

       

   > articles/views.py에서 C, U, D에 @login_required 데코레이터가 필요하나, D의 경우 @login_required와 @require_POST의 충돌로 문제가 발생. 

   ```python
   from django.contrib.auth.decorators import login_required
   from django.views.decorators.http import require_POST
   @login_required
   def create(request):
       ...
   
   @login_required
   def update(request, pk):
       ...
       
   #문제 해결을 위해 D의 경우 is_authenticated attribute 사용
   @require_POST
   def delete(request, pk):
       if request.user.is_authenticated:
           article = get_object_or_404(Article, pk=pk)
           article.delete()
       return redirect('articles:index')
   ```

   