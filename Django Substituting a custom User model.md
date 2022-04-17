## Substituting a custom User model

>User 모델 대체하기

- 일부 프로젝트에서는 Django의 내장 User 모델이 제공하는 인증 요구사항이 적절하지 않을 수 있다

  ex) username 대신 email을 식별 토큰으로 사용하는 사이트

- Django는 User를 참조하는데 사용하는 `AUTH_USER_MODEL `값을 제공, 재정의(override)할 수 있도록 함

- 새 프로젝트를 시작하는 경우, 기본 사용자 모델만으로 충분하더라도 커스텀 유저 모델을 설정하는 것을 강력하게 권장한다.

  - ##### 단, 프로젝트의 모든 migrations 혹은 첫 migrate를 실행하기 전에 이 작업을 마쳐야함



> `AUTH_USER_MODEL`

- User를 나타내는데 사용하는 모델
- 프로젝트가 진행되는 동안 변경할 수 없음
- 프로젝트 시작 시 설정하기 위한 것이며, 참조하는 모델은 첫번째 마이그레이션에서 사용할 수 있어야 함
- 기본갑 : 'auth.User' (auth 앱의 User 모델)



>Custom User 모델 정의하기

- 관리자 권한과 함께 완전한 기능을 갖춘 User 모델을 구현하는 기본 클래스인 `AbstractUser`를 상속받아 새로운 User 모델 작성

```python
#accounts/models.py
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    pass
```

- 기존에 Django가 사용하는 User 모델이었던 auth 앱의 User모델을 accounts 앱의 User 모델을 사용하도록 변경

```python
#settings.py
AUTH_USER_MODEL = 'accounts.User'
```

- admin site에 Custom User 모델 등록

```python
#accounts/admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User

admin.site.register(User, UserAdmin)
```

- 만약 프로젝트 중간에 Custom User 모델을 정의한다면?

  1. `db.sqlite3` 파일 삭제
  2. migrations 파일 모두 삭제(파일명에 숫자가 붙은 파일만)

  ```bash
  $ python manage.py makemigrations
  $ python manage.py migrate
  ```

--------



## Custom user & Built-in auth forms

- 기존에 User 모델에 연결되서 사용하던 form들은 Custom User 모델 설정시 사용이 불가능 해, 특정 form들은 Custom User 모델로 다시 연결해주는 작업이 필요.

> Custom Built-in Auth Forms

- 기존 User 모델을 사용하기 때문에 커스텀 User 모델로 다시 작성하거나 확장해야 하는 forms
  - UserCreationForm
  - UserChangeForm

```python
#accounts/forms.py
from django.contrib.auth.forms import UserChangeForm, UserCreationForm
from django.contrib.auth import get_user_model

class CustomUserChangeForm(UserChangeForm):

    # password = None

    class Meta:
        model = get_user_model() # User
        fields = ('email', 'first_name', 'last_name',)

class CustomUserCreationForm(UserCreationForm):
    
    class Meta(UserCreationForm.Meta):
        model = get_user_model() # User
        fields = UserCreationForm.Meta.fields + ('email',)
#UserCreationForm.Meta.fields은 ('username',)와 같음
```

> signup view 함수 코드 수정

```python
#accounts/views.py
from .forms import CustomUserChangeForm, CustomUserCreationForm

def signup(request):
    if request.user.is_authenticated:
        return redirect('articles:index')

    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            auth_login(request, user)
            return redirect('articles:index')
    else:
        form = CustomUserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
```

-----



