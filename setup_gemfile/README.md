# 젬파일의 구성

이 프로젝트에서 추가할 젬은 아래와 같다. `Gemfile`에 추가하고,

{%ace edit=false, lang='ruby', theme='monokai'%}
# Added for this project
gem 'jquery-turbolinks'
gem 'foundation-rails'
gem 'foundation-icons-sass-rails'
gem 'simple_form'
gem 'devise'
gem 'rolify'
gem 'authority'
gem "letter_opener", :group => :development
{%endace%}

번들 인스톨한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/bundle install
{%endace%}

`bin/bundle` 명령을 사용하면 `spring` 서버에서 관리하는 `bundle` 명령을 사용하게 되고, 그냥 `bundle` 명령을 사용하면 `spring`과 무관하게 실행된다. 따라서 `bin/bundle` 명령을 사용하여 실행속도가 빨라진 것을 체감하기 바란다.

이제 추가된 젬을 하나씩 설치해 보자.

지금까지 작업한 내용을 로컬 저장소로 커밋한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git add .
$ git commit -m "제02장 : 젬파일의 구성"
$ git tag "제02장"
{%endace%}

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장

---

_**References:**_

1. [Foundation Tools - Text Editor Snippets](http://foundation.zurb.com/develop/tools.html)
2. [zurb/foundation-5-sublime-snippets](https://github.com/zurb/foundation-5-sublime-snippets)
3. [plataformatec/simple_form](https://github.com/plataformatec/simple_form)
