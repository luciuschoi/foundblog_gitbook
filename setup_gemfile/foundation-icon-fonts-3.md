# foundation-icon-fonts-3 젬

`Foundation`에서 제공하는 [`Foundation Font 3`](http://zurb.com/playground/foundation-icon-fonts-3)를 사용하기 위해서는 `Sass`용으로 만들어진 젬을 사용하면 된다.

`Gemfile`에 아래와 같이 추가하고,

{%ace edit=false, lang='ruby'%}
gem 'foundation-icons-sass-rails'
{%endace%}

번들 인스톨한다.

{%ace edit=false, lang='sh'%}
$ bin/bundle install
{%endace%}

`app/assets/stylesheets/application.scss` 파일을 열고 아래와 같이 추가해 준다.

{%ace edit=false, lang='css'%}
@import 'foundation-icons';
{%endace%}


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장3절

---

_**Referenes:**_

1. [Foundation Font 3](http://zurb.com/playground/foundation-icon-fonts-3)
2. [zaiste/foundation-icons-sass-rails](https://github.com/zaiste/foundation-icons-sass-rails)
