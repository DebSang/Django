### **기본을 잊지말자**!!!!!!!!!!!!

##### 가상환경 설정 및 활성화

```bash
$ python -m venv venv
$ source venv/Scripts/Activate
```

##### 장고설치

```bash
$ pip install django
```

- 특정 버전 설치

  ```bash
  $ pip install django==3.2.12 #버전 기억하기!!!
  ```

- 설치 확인

  ```bash
  $ pip list #가상환경인지 아닌지 확인, 장고 등 프로그램이 설치되었는지 아닌지 확인!
  ```

---------

##### 프로젝트 생성 및 실행

> **[주의]**
>
> project 를 생성할 때, Python 이나 Django 에서 사용중인 이름은 피해야 한다. 
>
> 현재 폴더에 프로젝트를 생성해야 작업하기 편하므로 맨 뒤에 .을 꼭 붙인다!
>
> `-` 도 사용할 수 없다. (ex. django, test, class, django-test...)

```bash
$ django-admin startproject firstpjt .
$ python manage.py runserver
```



##### Application 생성

```bash
$ python manage.py startapp articles
```

> 일반적으로 Application명은 `복수형`으로 하는 것을 권장



##### Application 등록

> 반드시 **app 생성 후 등록** 순서를 지켜야한다.

- 방금 생성한 app을 사용하려면 프로젝트에 등록 해야 한다.

  ```python
  # settings.py
  
  INSTALLED_APPS = [
  	'articles',
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
  ]
  ```

  > INSTALLED_APPS의 app order
  >
  > ```python
  > INSTALLED_APPS = [
  >  # Local apps
  >  'blogs',
  > 
  >  # Third party apps
  >  'haystack',
  > 
  >  # Django apps
  >  'django.contrib.admin',
  >  'django.contrib.auth',
  >  'django.contrib.contenttypes',
  >  'django.contrib.sessions',
  >  'django.contrib.sites',
  > ]
  > ```



##### setting.py 설정

> 템플릿 상속

> https://docs.python.org/ko/3.9/library/pathlib.html#module-pathlib

- 템플릿 상속은 기본적으로 코드의 재사용성에 초점을 맞춤

- 템플릿 상속을 사용하면 사이트의 모든 공통 요소를 포함하고, 하위 템플릿이 재정의(override) 할 수있는 블록을 정의하는 기본 “skeleton” 템플릿을 만들 수 있음

  ```python
  # settings.py
  
  TEMPLATES = [
      {
          ...,
          'DIRS': [BASE_DIR / 'firstpjt' / 'templates'],
  ...
  ]
  ```

  - `app_name/templates` 디렉토리 외 추가 경로 설정



**`extends` tag**

- 자식(하위)템플릿이 부모 템플릿을 확장한다는 것을 알림

- 반드시 템플릿 최상단에 위치해야 함(== 템플릿의 첫번째 템플릿 태그여야 함)

  - 즉, 2개 이상 사용할 수 없음

  ```jinja2
  {% extends 'base.html' %}
  ```

  

**`block` tag**

- 하위 템플릿에서 재지정(overriden)할 수 있는 블록을 정의

- 하위 템플릿이 채울 수 있는 공간

- 가독성을 높이기 위해 선택적으로 `{% endblock %}` 태그에 이름 지정

  ```jinja
  {% block content %}
  {% endblock content %}
  ```



##### LANGUAGE_CODE

- 모든 사용자에게 제공되는 번역을 결정

- 이 설정이 적용 되려면 `USE_I18N`이 활성화되어 있어야 함

- http://www.i18nguy.com/unicode/language-identifiers.html

  ```python
  # settings.py
  
  LANGUAGE_CODE = 'ko-kr'
  ```

```
TIME_ZONE
```

- 데이터베이스 연결의 시간대를 나타내는 문자열 지정

- `USE_TZ`가 True이고 이 옵션이 설정된 경우 데이터베이스에서 날짜 시간을 읽으면 UTC 대신 새로 설정한 시간대의 인식 날짜&시간이 반환 됨

- `USE_TZ`이 False인 상태로 이 값을 설정하는 것은 error

- https://en.wikipedia.org/wiki/List_of_tz_database_time_zones

  ```plaintext
  # settings.py
  
  TIME_ZONE = 'Asia/Seoul'
  ```

