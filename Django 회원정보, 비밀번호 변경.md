## 0412 Workshop



##### 회원정보 수정

>accounts/urls.py

```python
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    path('update/', views.update, name='update'),
    ]
```



>그냥 `UserChangeForm`을 사용 시 일반 사용자가 접근해서는 안될 정보들 까지 모두 수정이 가능해짐
>
>따라서 `UserChangeForm`을 상속받아 `CustomUserChangeForm`이라는 서브 클래스를 작성해 접근 가능한 필드 조정
>
>accounts/forms.py

```python
from django.contrib.auth.forms import UserChangeForm
from django.contrib.auth import get_user_model


class CustomUserChangeForm(UserChangeForm):

    class Meta:
        model = get_user_model()
        fields = ('email', 'first_name', 'last_name') 
        #보여주기를 원하는 필드들만 넣기.
```

>get_user_model()

- 현재 프로젝트에서 활성화된 사용자 모델(active user model)을 반환

- Django는 User 클래스를 직접 참조하는 대신

  django.contrib.auth.get_user_model()을 사용하여 참조해야 하길 권장



>accounts/views.py

```python
from django.contrib.auth.forms import UserChangeForm
from .forms import CustomUserChangeForm

def update(request):
    if request.method == 'POST':
        form = CustomUserChangeForm(request.POST, instance=request.user)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = CustomUserChangeForm(instance=request.user)
    context = {
        'form' : form,
    }
    return render(request, 'accounts/update.html', context)
```



>accounts/update.html

```html
{% extends 'base.html' %}

{% block content %}
  <h1>회원정보수정</h1>
  <hr>
  <form action="{% url 'accounts:update' %}" method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit">
  </form>
  <a href="{% url 'articles:index' %}">back</a>
{% endblock content %}
```



>base.html
>
>로그인 시에만 회원정보수정이 노출되도록

```html
<body>
	<div class="container">
        <h3>Hello, {{ user }}</h3>
        {% if request.user.is_authenticated %}
			<a href="{% url 'accounts:update' %}">회원정보수정</a>
        {% else %}
        {% endif %}
        {% block content %}
        {% endblock content %}
    </div>
</body>
```

