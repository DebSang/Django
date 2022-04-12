## Authenticatiom System

> Django 인증 시스템은 django.contrib.auth에 Django contrib module로 제공(이미 settings.py의 INSTALLED_APPS에 저장되어있음)



##### accounts 앱 만들고 등록 및 url 설정

> 다른 이름으로 회원가입 앱을 만들어도 되지만, 편의성을 위해 accounts로 이름 짓는 것을 권장

```bash
$ python manage.py startapp accounts
```

```python
#settings.py
INSTALLED_APPS = [
    'articles',
    'accounts',
    ...,
]
```

```python
#crud/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
    path('accounts/', include('accounts.urls')),
]

# accounts/urls.py
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
]
```



##### admin 계정 만들기

>#회원가입 폼을 아직 안만들었을 경우 admin계정을 통해 로그인, 로그아웃 

```bash
$ python manage.py createsuperuser
```



##### 로그인

> 로그인은 session을 Create하는 로직과 같은데, 장고는 우리가 session의 메커니즘을 굳이 생각하지 않게끔 인증에 관한 built-in forms를 제공

- 'AuthenticationForm'
  - 사용자 로그인을 위한 form이며 request를 첫번째 인자로 취함
- login 함수
  - login(request, user, backend=None)
    - 현재 세션에 연결하려는 인증 된 사용자가 있는 경우 login()함수가 필요
    - 사용자를 로그인하며 view 함수에서 사용
    - HttpRequest 객체와 User객체가 필요
    - 로그인시, 세션에 user의 ID를 저장

>accounts/urls.py

```python
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    path('login/', views.login, name='login'),
    ]
```

>accounts/views.py

```python
from django.contrib.auth import login as auth_login #views의 login 함수와 구별하기 위해 기존의 login 함수를 auth_login으로 이름 변경
from django.contrib.auth.forms import AuthenticationForm #빌트인 폼
from django.shortcuts import render, redirect #render 및 redirect은 이후로 생략하겠음
from django.views.decorators.http import require_http_methods #데코레이터

@require_http_methods(['GET', 'POST'])
def login(request):
    if request.user.is_authenticated: #이미 로그인 되어 있는 상태인지 확인
        return redirect('articles:index') #로그인 되어 있다면 메인페이지로.
    if request.method == 'POST':
        form = AuthenticationForm(request, request.POST)
        if form.is_valid():
            auth_login(request, form.get_user()) #form 안에 user 정보가 있음
            return redirect('articles:index')
    else:
        form = AuthenticationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/login.html', context)
```

- get_user
  - AuthenticationForm의 인스턴스 메서드
  - 인스턴스의 유효성 검사를 통과했을 경우 로그인 한 사용자 객체로 할당, user를 제공

>accounts/login.html

```jinja2
{% extends 'base.html' %}

{% block content %}
  <h1>로그인</h1>
  <hr>
  <form action="" method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit">
  </form>
  <a href="{% url 'articles:index' %}">back</a>
{% endblock content %}
```

>base.html에 로그인 링크 작성 및 로그인 되어 있는 유저 정보 출력
>
>이미 로그인한 사람에게 로그인이 노출 안되도록 만들기.

```html
<body>
	<div class="container">
        <h3>Hello, {{ user }}</h3>
        {% if request.user.is_authenticated %}
        {% else %}
        	<a href="{%url 'accounts:login'%}">Login</a>
        {% endif %}
        {% block content %}
        {% endblock content %}
    </div>
</body>
```

