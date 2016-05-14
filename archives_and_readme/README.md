# Archives와 Readme 작성

월별로 글 목록을 볼 수 있는 페이지를 작성하기 위해서 상단 메뉴에는 이미 `Archives`라는 항목이 있다.

이를 위해서 우선 `posts` 컨트롤러(`app/controllers/posts_controller.rb`)에 아래와 같이 `archive` 액션을 추가하고,

{%ace edit=true, lang='ruby'%}
def archive
  @posts = Post.published_posts
end
{%endace%}

`routes.rb` 파일에서 아래와 같이 `resources :posts`에 `:collection`으로 등록한다.

{%ace edit=true, lang='ruby'%}
resources :posts do
  get 'archive', on: :collection
  get 'list_my', on: :collection
  resources :comments
end
{%endace%}

위와 같이 등록하면 `archive_posts_path` 메소드로 `posts#archive` 액션을 호출할 수 있는 `URI`를 지정할 수 있게 된다.

이제 이 액션에 대한 뷰 템플릿 파일(`app/views/posts/archive.html.erb`)을 생성하고 아래와 같이 작성한다.

{%ace edit=true, lang='rhtml'%}
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

그리고 여기서 사용한 `CSS` 클래스를 작성하기 위해 `app/assets/posts.css.scss` 파일에 아래와 같이 추가한다.

{%ace edit=true, lang='scss'%}
table#archives {
  color:#d9d9d9;
  margin-bottom:1em;
  width:100%;
  border:0;
  td {
    background-color: white;
  }
  td.year_month {
    font-size:1.5em;
    color:#c8c8c8;
    font-weight:bold;
    font-style:italic;
    padding-top:1em;
    padding-left:0;
  }
  td.day {
    text-align:center;
    border:1px solid #eaeaea;
    padding:5px;
    background-color: #d5ebef;
  }
  td.post {
    background-color: #f7f7f7;
  }
}
{%endace%}

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-26_19-12-40_zpsfa97002a.png)

자, 이제 상단 메뉴에 있는 `Readme` 항목의 뷰 파일을 만들기 위해서 `welcome` 컨트롤러와 `readme` 액션을 생성하자.

{%ace edit=true, lang='sh'%}
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
      create      app/assets/stylesheets/welcome.css.scss
{%endace%}

그리고 `readme` 액션에서는 특별한 작업이 필요없기 때문에 이 액션에 대한 뷰 파일(`app/views/welcome/readme.html.erb`)을 열고 아래와 같이 추가한다.

{%ace edit=true, lang='rhtml'%}
<h2>Readme</h2>
<br />

<%= image_tag "foundation_blog_emblem.png", class: 'emblem'  %>

<p>이 어플리케이션은 <code>Foundation 5</code>를 이용하여 레일스로 만든 블로그입니다. </p>
<p>"FoundBlog 따라하기"라는 Gitbook 책에서 데모로 작성한 블로그 프로젝트를 허로쿠로 배포하여 책의 내용을 이해하는데 도움을 주고자 하였습니다.</p>
<p>소스코드의 github 주소는 <code>https://github.com/LuciusChoi/foundblog</code>이며, 소스코드를 크론하기 위해서는 아래와 같이 터미널에서 실행하면 됩니다. </p>

<blockquote>
  $ git clone https://github.com/LuciusChoi/foundblog.git
</blockquote>
{%endace%}

물론 이 뷰 파일의 내용은 각자 원하는 내용으로 변경할 수 있다.

이제 관련 `CSS`를 추가하기 위해서 `app/assets/stylesheets/welcome.css.scss` 파일을 열고 아래와 같이 추가한다.

{%ace edit=true, lang='scss'%}
.emblem {
  float:left;
  width:30%;
  margin-right:1em;
}
{%endace%}

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-26_19-16-58_zps96d40a36.png)


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제11장
