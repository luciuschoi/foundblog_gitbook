# Turbolinks 젬의 보완

레일스 4.0부터 디폴트로 지원하기 시작한 `Turbolinks`는 애플리케이션 내의 링크를 보다 빠르게 연결되도록 해 준다. 이 기술은 `PJAX` 기술로부터 가져온 것으로 `PJAX`와의 차별점은, `PJAX`가 특정 요소의 값만은 서버로부터 비동기적으로 가져오는 반면, `Turbolinks`는 특정 페이지의 `Title`과 `Body` 태그 값만을 `Ajax`로 갱신되도록 해 준다.

그러나, `Turbolinks`를 사용할 경우 페이지 이동시 `$(document).ready` 함수가 제대로 동작하지 않게 되는 문제점이 발생한다.

이부분에 대한 자세한 내용은 이 책의 범주를 벗어나는 것이므로 여기서는 위에서 언급한 문제를 해결하기 위해서 `jquery.turbolinks` 젬을 사용하기로 한다.

> #### Note::참고
>
> `jquery.turblinks` 젬을 사용하는 것 외에도 여러가지 해결책이 있지만, 특히, [Brandon Hilkert의 글](http://brandonhilkert.com/blog/organizing-javascript-in-rails-application-with-turbolinks/)이 도움이 되었다.


처음부터 작업을 따라 해온 경우라면 이미 `Gemfile` 파일에 아래와 같이 젬을 추가한 후 번들 인스톨한 상태다.

{%ace edit=false, lang='ruby', theme='monokai'%}
gem 'jquery-turbolinks'
{%endace%}

`juqery.turbolinks`는 `application.js` 파일에서 추가하는 위치가 중요한다. 즉, 상단의 `//= require jquery` 바로 아래에 추가해 주어야 하고, `//= require turbolinks`는 제일 아래로 위치시키야 한다. 그리고 기타 다른 자바스크립트들은 그 사이에 둔다.

{%ace edit=false, lang='javascript', theme='monokai'%}
//= require jquery
//= require jquery.turbolinks
//= require jquery_ujs
//
// ... your other scripts here ...
//
//= require turbolinks
{%endace%}

따라서 `app/assets/javascripts/application.js` 파일을 열고 아래와 같은 순서로 변경한다.

{%ace edit=false, lang='javascript', theme='monokai'%}
//= require jquery
//= require jquery.turbolinks
//= require jquery_ujs
//= require foundation
//= require_tree .
//= require turbolinks
{%endace%}

> #### Caution::주의
>
> 자바스크립트 파일간의 의존성이 문제가 될 때는 `require_tree .` 지시어는 삭제하고 의존성 순서대로 자바스크립트 파일을 일일이 추가해 주어야 한다.


지금까지 작업한 내용을 로컬 저장소로 커밋한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git add .
$ git commit -m "제02장 1절 : Turbolinks 젬의 보완"
$ git tag "제02장1절"
{%endace%}

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장1절

---

_**References:**_

1. [rails/turbolinks](https://github.com/rails/turbolinks)
2. [Introducing Turbolinks for Rails 4.0](http://geekmonkey.org/articles/28-introducing-turbolinks-for-rails-4-0)
2. [jquery.turbolinks](https://github.com/rails/turbolinks#jqueryturbolinks)
3. [Rails 4: how to use $(document).ready() with turbo-links](http://stackoverflow.com/questions/18770517/rails-4-how-to-use-document-ready-with-turbo-links)
4. [Railscasts #390 Turbolinks](http://railscasts.com/episodes/390-turbolinks)
5. [Organizing Javascript In Rails Application With Turbolinks](http://brandonhilkert.com/blog/organizing-javascript-in-rails-application-with-turbolinks/)
