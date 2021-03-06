# simple_form 젬

[`simple_form`](https://github.com/plataformatec/simple_form) 젬은 레일스 폼 헬퍼메소드인 [`form_for`](http://guides.rubyonrails.org/form_helpers.html)를 대신해서 사용할 수 있는 `simple_form_for` 헬퍼 메소드를 제공해 준다. 이 메소드를 사용하면 훨씬 간단한 옵션을 지정하여 원하는 형태의 폼을 생성할 수 있다. 또한, `simple_form`을 인스톨할 때 `bootstrap`이나 `foundation`에 대한 옵션을 지정하면 이들 프레임워크와 잘 융화하도록 `html`을 렌더링해 준다.

처음부터 작업을 따라 해온 경우라면 이미 `Gemfile` 파일에 아래와 같이 젬을 추가한 후 번들 인스톨한 상태다.

{%ace edit=false, lang='ruby', theme='monokai'%}
gem 'simple_form'
{%endace%}

아래와 같이 `simple_form` 젬을 셋업할 때 옵션도 함께 지정하는 것을 기억하기 바란다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/rails g simple_form:install --foundation
Running via Spring preloader in process 93551
      create  config/initializers/simple_form.rb
      create  config/initializers/simple_form_foundation.rb
       exist  config/locales
      create  config/locales/simple_form.en.yml
      create  lib/templates/erb/scaffold/_form.html.erb
{%endace%}

마지막에 생성된 `_form.html.erb` 파일을 에디터로 열어 `f.button`에 아래와 같이 클래스를 추가한다. 이것은 레일스의 `scaffold` 제너레이터로  폼 파일을 생성할 때 디폴트로 적용된다.

{%ace edit=false, lang='rhtml', theme='monokai'%}
<div class="form-actions">
  <%%= f.button :submit, class: 'button small' %>
</div>
{%endace%}

지금까지 작업한 내용을 로컬 저장소로 커밋한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git add .
$ git commit -m "제02장 4절 : simple_form 젬"
$ git tag "제02장4절"
{%endace%}


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장4절

---

_**References:**_

1. [plataformatec/simple_form](https://github.com/plataformatec/simple_form)
