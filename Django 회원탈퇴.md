##### 회원탈퇴

>accounts/urls.py

```python
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    path('delete/', views.delete, name='delete'),
    ]
```

>accounts/views.py

```python
from django.contrib.auth import logout as auth_logout
from django.views.decorators.http import require_POST

@require_POST
def delete(request):
    if request.user.is_authenticated:
        #세션 데이터를 지우려면 로그아웃을 해야하고
        #반드시 회원탈퇴 후 로그아웃 함수 호출
        request.user.delete()
        auth_logout(request)
    return redirect('articles:index')
```

>base.html

```html
<body>
	<div class="container">
        <h3>Hello, {{ user }}</h3>
        {% if request.user.is_authenticated %}
        	<form action="{% url 'accounts:delete' %}">
        		{% csrf_token %}
        		<input type="submit" value="회원탈퇴">
      		</form>
        {% else %}
        	<a href="{%url 'accounts:login'%}">Login</a>
      		<a href="{% url 'accounts:signup' %}">Signup</a>
        {% endif %}
        {% block content %}
        {% endblock content %}
    </div>
</body>
```

