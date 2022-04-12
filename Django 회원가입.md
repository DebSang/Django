##### 회원가입

>accounts/urls.py

```python
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    path('signup/', views.signup, name='signup'),
    ]
```

>accounts/views.py

```python
from django.contrib.auth.forms import UserCreationForm #빌트인 폼
from django.contrib.auth import login as auth_login

@require_http_methods(['GET', 'POST'])
def signup(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid:
            user = form.save()
            auth_login(request, user) #회원가입 후 바로 로그인하기
            return redirect('articles:index')
    else:
        form = UserCreationForm()
    context = {
        'form' : form
    }
    return render(request, 'accounts/signup.html', context)
```

>accounts/signup.html

```jinja2
{% extends 'base.html' %}

{% block content %}
  <h1>회원가입</h1>
  <hr>
  <form action="{% url 'accounts:signup' %}" method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit">
  </form>
  <a href="{% url 'articles:index' %}">back</a>
{% endblock content %}
```

>이미 로그인한 사람에게 회원가입이 노출 안되도록 만들기.
>
>base.html

```html
<body>
	<div class="container">
        <h3>Hello, {{ user }}</h3>
        {% if request.user.is_authenticated %}
        {% else %}
        	<a href="{%url 'accounts:login'%}">Login</a>
      		<a href="{% url 'accounts:signup' %}">Signup</a>
        {% endif %}
        {% block content %}
        {% endblock content %}
    </div>
</body>
```

