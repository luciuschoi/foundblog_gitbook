# foundation-icons-sass-rails 젬

`Foundation`에서 제공하는 [`Foundation Font 3`](http://zurb.com/playground/foundation-icon-fonts-3)를 사용하기 위해서는 `Sass`용으로 만들어진 젬(foundation-icons-sass-rails)을 사용하면 된다.

처음부터 작업을 따라 해온 경우라면 이미 `Gemfile` 파일에 아래와 같이 젬을 추가한 후 번들 인스톨한 상태다.

{%ace edit=false, lang='ruby', theme='monokai'%}
gem 'foundation-icons-sass-rails'
{%endace%}

이제 `app/assets/stylesheets/application.scss` 파일을 열고 아래와 같이 추가한다.

{%ace edit=false, lang='css', theme='monokai'%}
@import 'foundation_and_overrides';
@import 'foundation-icons';
{%endace%}


지금까지 작업한 내용을 로컬 저장소로 커밋한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git add .
$ git commit -m "제02장 3절 : foundation-icons-sass-rails 젬"
$ git tag "제02장3절"
{%endace%}


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장3절

---

_**Referenes:**_

1. [Foundation Font 3](http://zurb.com/playground/foundation-icon-fonts-3)
2. [zaiste/foundation-icons-sass-rails](https://github.com/zaiste/foundation-icons-sass-rails)
