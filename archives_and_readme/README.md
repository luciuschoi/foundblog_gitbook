# Archives와 Readme 작성

월별로 글 목록을 볼 수 있는 페이지로 연결하도록 상단 메뉴에는 `Archives`라는 메뉴 항목이 있다.

이를 위해서 우선 `posts` 컨트롤러(`app/controllers/posts_controller.rb`)에 아래와 같이 `archive` 액션을 추가하고,

{%ace edit=false, lang='ruby', theme='monokai'%}
def archive
  @posts = Post.published_posts
end
{%endace%}

`routes.rb` 파일에서 `resources :posts`에 `:collection`으로 등록한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
resources :posts do
  get 'archive', on: :collection
  get 'list_my', on: :collection
  resources :comments
end
{%endace%}

위와 같이 등록하면 `archive_posts_path` 메소드로 `posts#archive` 액션을 호출할 수 있는 `URI`를 지정할 수 있게 된다.

이제 이 액션에 대한 뷰 템플릿 파일(`app/views/posts/archive.html.erb`)을 생성하고 아래와 같이 작성한다.

{%ace edit=false, lang='rhtml', theme='monokai'%}
<h2>Archives</h2>
<table id='archives'>
  <% archive_year_month = "" %>
  <% @posts.each do | post | %>
    <% post_month = post.created_at.strftime("%Y-%m") %>
    <% if archive_year_month != post_month %>
    <tr>
      <td colspan='2' class='year_month'>
        <%= icon('calendar') %>
        <%= archive_year_month = post_month %>
      </td>
    </tr>
    <% end %>
    <tr>
      <td class='day'><%=post.created_at.day%></td>
      <td class='post'>
        <%= link_to "#{truncate(post.title)} (#{post.comments.size}), #{post.user.email}".html_safe, posts_path(post) %>
      </td>
    </tr>
  <% end %>
</table>
{%endace%}

그리고 여기서 사용한 `CSS` 클래스를 작성하기 위해 `app/assets/posts.scss` 파일에 아래와 같이 추가한다.

{%ace edit=false, lang='scss', theme='monokai'%}
table#archives {
  margin-bottom:1em;
  width:100%;
  td.year_month {
    border: 2px solid white;
    font-size:1.2em;
    color:#aaa;
    font-weight:bold;
    font-style:italic;
    padding-left: 0;
  }
  td.day {
    border: 2px solid white;
    text-align:center;
    padding:5px;
    background-color: #d5ebef;
  }
  td.post {
    border: 2px solid white;
    background-color: #f7f7f7;
  }
}
{%endace%}

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-26_19-12-40_zpsfa97002a.png)

자, 이제 상단 메뉴에 있는 `Readme` 항목의 뷰 파일을 만들기 위해서 `welcome` 컨트롤러와 `readme` 액션을 생성한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/rails g controller welcome readme
      create  app/controllers/welcome_controller.rb
       route  get 'welcome/readme'
      invoke  erb
      create    app/views/welcome
      create    app/views/welcome/readme.html.erb
      invoke  test_unit
      create    test/controllers/welcome_controller_test.rb
      invoke  helper
      create    app/helpers/welcome_helper.rb
      invoke    test_unit
      create      test/helpers/welcome_helper_test.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/welcome.js.coffee
      invoke    scss
      create      app/assets/stylesheets/welcome.scss
{%endace%}

그리고 `readme` 액션에서는 특별한 작업이 필요없기 때문에 이 액션에 대한 뷰 파일(`app/views/welcome/readme.html.erb`)을 열고 아래와 같이 추가한다.

{%ace edit=false, lang='rhtml', theme='monokai'%}
<h2>Readme</h2>
<br />

<%= image_tag "foundation_blog_emblem.png", class: 'emblem' %>

<p>이 애플리케이션은 <code>Foundation 6</code>를 이용하여 레일스로 만든 블로그입니다. </p>
<p>"FoundBlog 따라하기"라는 Gitbook 책에서 데모로 작성한 블로그 프로젝트를 허로쿠로 배포하여 책의 내용을 이해하는데 도움을 주고자 하였습니다.</p>
<p>소스코드의 github 주소는 <code><a href='https://github.com/luciuschoi/foundblog_app' target="_blank">https://github.com/luciuschoi/foundblog_app</a></code>이며, 소스코드를 크론하기 위해서는 아래와 같이 터미널에서 실행하면 됩니다. </p>

<div style="clear:both;">
  <blockquote class='code'>
    $ git clone https://github.com/luciuschoi/foundblog_app.git
  </blockquote>
</div>
{%endace%}

물론 이 뷰 파일의 내용은 각자 원하는 내용으로 변경할 수 있다.

이제 관련 `CSS`를 추가하기 위해 `app/assets/stylesheets/welcome.scss` 파일을 열고 아래와 같이 추가한다.

{%ace edit=false, lang='scss', theme='monokai'%}
.emblem {
  float:left;
  width:25%;
  margin-right:1em;
  margin-bottom:1em;
}

blockquote.code {
  border-left: 3px solid #cacaca;
  padding: .5rem 1rem;
  background-color: #eaeaea;
  font-family: 'courier';
}
{%endace%}

그리고, `app/assets/stylesheets/application.scss` 파일을 열고 추가한다.

{%ace edit=false, lang='scss', theme='monokai'%}
...
@import "welcome";
...
{%endace%}

`config/routes.rb` 파일을 열고, `welcome#readme` 액션에 대한 라우트를 아래와 같이 업데이트한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
...
get 'welcome/readme', as: :readme
...
{%endace%}

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-26_19-16-58_zps96d40a36.png)


마지막으로 `app/views/layouts/application.html.erb` 파일을 열고, 상단 메뉴에 링크를 추가해 준다.

{%ace edit=false, lang='rhtml', theme='monokai'%}
...
<div class="top-bar" id="example-menu">
  <div class="top-bar-left">
    <ul class="vertical medium-horizontal menu">
      <li class="menu-text">Found<i>Blog</i>
      </li>
      <li>
          <a href="/">HOME</a>
      </li>
      <li><%= link_to "Archives", archive_posts_path %></li>
      <li><%= link_to "Readme", readme_path %></li>
    </ul>
...    
{%endace%}


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제11장
