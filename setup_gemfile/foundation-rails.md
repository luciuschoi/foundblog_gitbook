# foundation-rails 젬

[`foundation-rails`](https://github.com/zurb/foundation-rails)젬은 `Foundation`을 레일스에서 사용하기 편리하게 만들어 놓은 젬이다. 현재 이 젬은 `Foundation`의 최선버전인 `6`를 지원한다.

> #### Note::노트
>
> `Atom` 에디터를 사용할 경우 관련 플로그인([Zurb Foundation for sites support for Atom.io](https://atom.io/packages/atom-zurb-foundation))을 설치하면 편리하게 코딩할 수 있다.

이제 `Foundation 6`를 프로젝트에 추가하는 작업이 필요한다. 이것은 레일스의 `asset pipeline`에서 사용할 수 있도록 하는 조치이며 아래와 같은 명령으로 인스톨한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/rails g foundation:install
Running via Spring preloader in process 92906
      insert  app/assets/javascripts/application.js
      append  app/assets/javascripts/application.js
      create  app/assets/stylesheets/foundation_and_overrides.scss
      create  app/assets/stylesheets/_settings.scss
      insert  app/assets/stylesheets/application.css
    conflict  app/views/layouts/application.html.erb
Overwrite /Users/hyo/prj/r5/foundblog_app/app/views/layouts/application.html.erb? (enter "h" for help) [Ynaqdh] Y
       force  app/views/layouts/application.html.erb
   identical  app/assets/stylesheets/foundation_and_overrides.scss
   identical  app/assets/stylesheets/_settings.scss   
{%endace%}

[기존의 디폴트 애플리케이션 레이아웃 파일]

{%ace edit=false, lang='rhtml', theme='monokai'%}
<!DOCTYPE html>
<html>
<head>
  <title>FoundblogApp</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<%= yield %>

</body>
</html>
{%endace%}

[Foundation 인스톨이 적용된 애플리케이션 레아이웃 파일]

{%ace edit=false, lang='rhtml', theme='monokai'%}
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <title><%= content_for?(:title) ? yield(:title) : "Untitled" %></title>

    <%= stylesheet_link_tag    "application" %>
    <%= javascript_include_tag "application", 'data-turbolinks-track' => true %>
    <%= csrf_meta_tags %>
  </head>

  <body>

    <%= yield %>

  </body>
</html>
{%endace%}

그러나 디폴트 애플리케이션 레이아웃 파일의 중요한 부분들이 `Foundation`용 레이아웃으로 덮어 쓰여진 파일에서 빠진 것을 알 수 있다. 따라서 이 두개의 레이아웃을 합쳐서 아래와 같이 변경한다.

{%ace edit=false, lang='rhtml', theme='monokai'%}
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <title><%= content_for?(:title) ? yield(:title) : "FoundblogApp" %></title>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
    <%= javascript_include_tag "application", 'data-turbolinks-track' => true %>
    <%= csrf_meta_tags %>
  </head>

  <body>

    <%= yield %>

  </body>
</html>
{%endace%}


레일스에서는 디폴트로 `SCSS`를 사용하므로 추가적인 작업이 필요한다. 즉, 다른 `.scss` 파일에서도 `Foundation 6`의 클래스, mixin, 변수들을 사용할 수 있도록 하기 위해서는 `application.css` 파일을 `application.scss`로 이름을 변경하고 아래와 같이 설치시 생성된 `foundation_and_overrides.scss` 파일을 임포트한다.

{%ace edit=false, lang='rhtml', theme='monokai'%}
@import 'foundation_and_overrides';
{%endace%}

`Motion UI`은 `UI` 트랜지션과 애니메이션을 만들어주는 Sass 라이브러리인데, 이를 사용할 경우에는 `foundation_and_overrides.scss` 파일에서 아래의 코드라인을 찾아 코멘트 문자를 제거해 준다.

{%ace edit=false, lang='scss', theme='monokai'%}
// @import 'motion-ui/motion-ui';
// @include motion-ui-transitions;
// @include motion-ui-animations;
{%endace%}

이제부터는 추가로 작성하는 `.scss` 파일은 이 파일에 `@import` 해 주어야 한다. 예를 들어 `posts.scss`파일에 내용을 추가한 후 반영하기 위해서는 아래와 같이 임포트해 주어야 한다.

{%ace edit=false, lang='rhtml', theme='monokai'%}
@import 'foundation_and_overrides';
{%endace%}

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장2절

---


_**References:**_

1. [Foundation Tools - Text Editor Snippets](http://foundation.zurb.com/develop/tools.html)
2. [zurb/foundation-5-sublime-snippets](https://github.com/zurb/foundation-5-sublime-snippets)
3. [Zurb Foundation for sites support for Atom.io](https://atom.io/packages/atom-zurb-foundation)
