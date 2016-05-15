# Turbolinks 젬의 보완

눈치 챈 독자도 있을 것이지만, 블로그 화면의 오른쪽 상단에 있는 `dropdown` 메뉴가 제대로 동작하지 않는다. 마우스 오버하면 하위메뉴가 보여야 한다.

레일스 4.0부터 디폴트로 지원하기 시작한 `Turbolinks`는 어플리케이션 내의 링크를 보다 빠르게 연결되도록 한 것이다. 이 기술은 PJAX 기술로부터 가져온 것으로 PJAX와의 차별점은, PJAX가 특정 요소의 값만은 서버로부터 비동기적으로 가져오는 반면, Turbolinks는 특정 페이지의 `Title`과 `Body` 태그 값만은 Ajax로 갱신하게 되는 것이다.

이부분에 대한 자세한 내용은 이 책의 범주를 벗어나는 것이므로 여기서는 위에서 언급한 문제를 해결하기 위해서 `jquery.turbolinks` 젬을 사용하기로 한다. `Gemfile` 파일에 아래와 같이 젬을 추가하고,

{%ace edit=true, lang='ruby'%}
gem 'jquery-turbolinks'
{%endace%}

그리고 아래와 같이 번들 인스톨한 후, 어플리케이션을 다시 실행한다.

{%ace edit=true, lang='sh'%}
$ bin/bundle install
{%endace%}
`juqery.turbolinks`는 `application.js` 파일에서 추가하는 위치가 중요한다. 즉, 상단의 `//= require jquery` 바로 아래에 추가해 주어야 하고, `//= require turbolinks`는 제일 아래로 위치시키야 한다. 그리고 기타 다른 자바스크립트들은 그 사이에 두어야 한다.

{%ace edit=true, lang='javascript'%}
//= require jquery
//= require jquery.turbolinks
//= require jquery_ujs
//
// ... your other scripts here ...
//
//= require turbolinks
{%endace%}

따라서 `app/assets/javascripts/application.js` 파일을 열고 아래와 같은 순서로 변경한다.

{%ace edit=true, lang='javascript'%}
//= require jquery
//= require jquery.turbolinks
//= require jquery_ujs
//= require foundation
//= require_tree .
//= require turbolinks

$(function(){ $(document).foundation(); });
{%endace%}

이제 제대로 동작해야 한다.

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제06장

---

_**References:**_

1. [rails/turbolinks](https://github.com/rails/turbolinks)
2. [Introducing Turbolinks for Rails 4.0](http://geekmonkey.org/articles/28-introducing-turbolinks-for-rails-4-0)
2. [jquery.turbolinks](https://github.com/rails/turbolinks#jqueryturbolinks)
3. [Rails 4: how to use $(document).ready() with turbo-links](http://stackoverflow.com/questions/18770517/rails-4-how-to-use-document-ready-with-turbo-links)
4. [Railscasts #390 Turbolinks](http://railscasts.com/episodes/390-turbolinks)
