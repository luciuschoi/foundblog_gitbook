# foundation-icon-fonts-3 젬

`Foundation`에서 제공하는 [`Foundation Font 3`](http://zurb.com/playground/foundation-icon-fonts-3)를 사용하기 위해서는 `Sass`용으로 만들어진 젬을 사용하면 된다. `Gemfile`에 아래와 같이 추가하고,

{%ace edit=true, lang='ruby'%}
gem 'foundation-icons-sass-rails'
{%endace%}

번들 인스톨한다.

{%ace edit=true, lang='sh'%}
$ bin/bundle install
{%endace%}

`app/assets/stylesheets/application.scss` 파일을 열고 아래와 같이 추가해 준다.

{%ace edit=true, lang='css'%}
@import 'foundation-icons';
{%endace%}

그리고 불린(boolean) 속성을 아이콘으로 표시하기 위해서 `application_helper.rb` 파일에 `published_icon`이라는 헬퍼 메소드를 하나 추가하자.

{%ace edit=true, lang='ruby'%}
def published_icon(boolean)
  if boolean
    "<i class='fi-check'></i> 작성완료".html_safe
  else
    "<i class='fi-pencil'></i> 작성중...".html_safe
  end
end
{%endace%}


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제05장

---

_**Referenes:**_

1. [Foundation Font 3](http://zurb.com/playground/foundation-icon-fonts-3)
2. [zaiste/foundation-icons-sass-rails](https://github.com/zaiste/foundation-icons-sass-rails)