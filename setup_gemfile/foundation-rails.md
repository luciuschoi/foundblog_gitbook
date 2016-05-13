# foundation-rails 젬

[`foundation-rails`](https://github.com/zurb/foundation-rails)젬은   `Foundation`을 레일스에서 사용하기 편리하게 만들어 놓은 젬이다. 현재 이 젬은 `Foundation`의 최선버전인 `5`를 지원한다.

> #### Note::노트
> 
> `Sublime Text Editor`에서 [`foundation-5-sublime-snippets`](https://github.com/zurb/foundation-5-sublime-snippets) 패키지를 사용하면 `Foundation`에서 사용하는 전용 컴포넌트를 손쉽게 추가할 수 있어 도움이 된다.

이제 `Foundation 5`를 프로젝트에 추가하는 작업이 필요한다. 이것은 레일스의 `asset pipeline`에서 사용할 수 있도록 하는 조치이며 아래와 같은 명령으로 인스톨한다.

```bash
$ bin/rails g foundation:install
      insert  app/assets/javascripts/application.js
      append  app/assets/javascripts/application.js
      create  app/assets/stylesheets/foundation_and_overrides.scss
      append  app/assets/stylesheets/foundation_and_overrides.scss
      insert  app/assets/stylesheets/application.css
    conflict  app/views/layouts/application.html.erb
Overwrite /Users/hyo/prj/r4/FoundBlog/app/views/layouts/application.html.erb? (enter "h" for help) [Ynaqdh] Y
       force  app/views/layouts/application.html.erb
```

[기존의 디폴트 어플리케이션 레이아웃 파일]

```html
<!DOCTYPE html>
<html>
<head>
  <title>FoundBlog</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<%= yield %>

</body>
</html>
```

[Foundation 인스톨이 적용된 어플리케이션 레아이웃 파일]

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <title><%= content_for?(:title) ? yield(:title) : "foundation-rails" %></title>

    <%= stylesheet_link_tag    "application" %>
    <%= javascript_include_tag "vendor/modernizr" %>
    <%= csrf_meta_tags %>
  </head>

  <body>

    <%= yield %>
    <%= javascript_include_tag "application" %>
  </body>
</html>
```

그러나 디폴트 어플리케이션 레이아웃 파일의 중요한 부분들이 `Foundation`용 레이아웃으로 덮어 쓰여진 파일에서 빠진 것을 알 수 있다. 따라서 이 두개의 레이아웃을 합쳐서 아래와 같이 변경한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><%= content_for?(:title) ? yield(:title) : "FoundBlog" %></title>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
    <%= javascript_include_tag "vendor/modernizr" %>
    <%= favicon_link_tag 'favicon.ico' %>
    <%= csrf_meta_tags %>
  </head>
  <body>

  <%= yield %>

  </body>
</html>
```

레일스에서는 디폴트로 `Sass`를 사용하므로 추가적인 작업이 필요한다. 즉, 다른 `.scss` 파일에서도 `Foundation 5`의 클래스, mixin, 변수들을 사용할 수 있도록 하기 위해서는 `application.css` 파일을 `application.scss`로 이름을 변경하고 아래와 같이 설치시 생성된 `foundation_and_overrides.scss` 파일을 임포트한다.

```html
@import 'foundation_and_overrides';
```

이제부터는 추가로 작성하는 `.scss` 파일은 이 파일에 `@import` 해 주어야 한다. 예를 들어 `posts.css.scss`파일에 내용을 추가한 후 반영하기 위해서는 아래와 같이 임포트해 주어야 한다.

```html
@import 'foundation_and_overrides';
@import 'posts';
```

---

> **소스보기** https://github.com/LuciusChoi/foundblog/tree/제2.1장

---


_**References:**_

1. [Foundation Tools - Text Editor Snippets](http://foundation.zurb.com/develop/tools.html)
2. [zurb/foundation-5-sublime-snippets](https://github.com/zurb/foundation-5-sublime-snippets)
