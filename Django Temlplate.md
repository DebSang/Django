[TOC]

# Template

> https://docs.djangoproject.com/en/3.2/topics/templates/#module-django.template
>
> 데이터 표현을 제어하는 도구이자 표현에 관련된 로직

<br>

## Django template language

> https://docs.djangoproject.com/en/3.2/ref/templates/language/
>
> https://docs.djangoproject.com/ko/3.2/ref/templates/builtins/#built-in-template-tags-and-filters

- django template에서 사용하는 built-in template system
- 조건, 반복, 변수 치환, 필터 등의 기능을 제공
- Django 템플릿 시스템은 단순히 Python이 HTML에 포함 된 것이 아님
  - 프로그래밍적 로직이 아니라 프레젠테이션을 표현하기 위한 것
- 파이썬처럼 일부 프로그래밍 구조(if, for 등)를 사용할 수 있지만 이건 해당 Python 코드로 실행되는 것이 아님

<br>

**syntax**

1. variables

- `{{ variables }}`

- 변수명은 영,숫자와 밑줄(_)의 조합으로 구성될 수 있으나 밑줄로는 시작 할 수 없음
- 변수명에 공백이나 구두점 문자를 사용할 수 없음
- `dot(.)`를 사용하여 변수 속성에 접근

<br>

2. Filters

- `{{ variable|filter }}`
- 표시할 변수를 수정
- 파이프(|)를 사용하여 적용

<br>

3. Tags

- `{% tag %}`
- 출력 테스트를 만들거나 반복 또는 논리를 수행하여 제어 흐름을 만드는 등 보다 복잡한 일들을 수행
- 일부 태그는 시작과 종료 태그가 필요(`{% tag %}` ... `{% endtag %}`)

<br>

4. Comments

- `{# lorem #}`

<br>

## Template inheritance

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

<br>

**`extends` tag**

- 자식(하위)템플릿이 부모 템플릿을 확장한다는 것을 알림
- 반드시 템플릿 최상단에 위치해야 함(== 템플릿의 첫번째 템플릿 태그여야 함)
  - 즉, 2개 이상 사용할 수 없음

<br>

**`block` tag**

- 하위 템플릿에서 재지정(overriden)할 수 있는 블록을 정의

- 하위 템플릿이 채울 수 있는 공간

- 가독성을 높이기 위해 선택적으로 `{% endblock %}` 태그에 이름 지정

  ```django
  {% block content %}
  {% endblock content %}
  ```

<br>

**Template system**

>https://docs.djangoproject.com/ko/3.1/misc/design-philosophies/#template-system

1. 표현과 로직(view)을 분리

- 우리는 템플릿 시스템이 표현을 제어하는 도구이자 표현에 관련된 로직일 뿐이라고 봅니다. 
- 템플릿 시스템은 이러한 기본 목표를 넘어서는 기능을 지원하지 말아야 합니다.

<br>

2. 중복을 배제

- 대다수의 동적 웹사이트는 공통 헤더, 푸터, 네이게이션 바 같은 사이트 공통 디자인을 갖습니다. 
- Django 템플릿 시스템은 이러한 요소를 한 곳에 저장하기 쉽게 하여 중복 코드를 없애야 합니다. 

<br>

## HTML Form

**HTML `<form>` element**

- 웹에서 사용자 정보를 입력하는 여러 방식(text, button, checkbox, file, hidden, image, password, radio, reset, submit)을 제공하고, 사용자로부터 할당된 데이터를 서버로 전송하는 역할을 담당
- 핵심 속성
  - action : 입력 데이터가 전송될 URL 지정
  - method : 입력 데이터 전달 방식 지정

<br>

**HTML `<input>` element**

- 사용자로부터 데이터를 입력 받기 위해 사용
- `type` 속성에 따라 동작 방식이 달라짐
- 핵심 속성
  - **name**
  - 중복 가능, 양식을 제출했을 때 name이라는 이름에 설정된 값을 넘겨서 값을 가져올 수 있음
  - 주요 용도는 GET/POST 방식으로 서버에 전달하는 파라미터(name 은 key , value 는 value)로 `?key=value&key=value` 형태로 전달

<br>

### HTTP request methods

**HTTP**

- HTML 문서와 같은 리소스들을 가져올 수 있도록 해주는 프로토콜(규칙, 규약)
- 웹에서 이루어지는 모든 데이터 교환의 기초
- HTTP는 주어진 리소스가 수행 할 원하는 작업을 나타내는 request methods를 정의

<br>

**GET**

- 서버로부터 **정보를 조회**하는 데 사용
- 데이터를 가져올 때만 사용해야 함
- 데이터를 서버로 전송할 때 body가 아닌 **Query String Parameters**를 통해 전송

<br>

**throw & catch**

- throw

  ```python
  # first_project/urls.py
  
  path('throw/', views.throw),
  ```

  ```python
  # articles/views.py 
  
  def throw(request):
      return render(request, 'throw.html')
  ```

  ```html
  <!-- articles/templates/throw.html -->
  
  <form action="/catch/" method="GET">
    <label for="message">Throw</label>
    <input type="text" id="message" name="message">
    <input type="submit">
  </form>
  ```

- catch

  ```python
  # first_project/urls.py
  
  path('catch/', views.catch),
  ```

  ```python
  # articles/views.py
  
  def catch(request):
      message = request.GET.get('message')
      context = {
          'message': message,
      }
      return render(request, 'catch.html', context)
  ```

  ```django
  <!-- articles/templates/catch.html -->
  
  <h1>너가 던져서 내가 받은건 {{ message }}야!</h1>
  <a href="/throw/">뒤로</a>
  ```

<br>

**Request object** 

> https://docs.djangoproject.com/en/3.1/ref/request-response/#module-django.http

- 요청 간의 모든 정보를 담고 있는 변수
- 페이지가 요청되면 Django는 요청에 대한 메타 데이터를 포함하는 `HttpRequest` 객체를 만들고
- 그런 다음 Django는 적절한 view 함수를 로드하고 `HttpRequest`를 뷰 함수의 첫 번째 인수로 전달. 
- 그리고 각 view는 `HttpResponse` 개체를 반환한다.

