# 블로그에 검색기능 추가하기

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-20_19-55-30_zps5fa425a2.png)

[www.ruby-toolbox.com](https://www.ruby-toolbox.com/categories/rails_search)를 검색하면 위의 캡쳐화면에서와 같이 `Sunspot`을 가장 많이 사용하고 있다. 거의 같은 빈도로 많이 사용하는 것이 `SunspotRails`인데 이것은 `Sunspot`을 `AcitveRecord`와 연계해서 사용할 수 있도록 한 것이다.

최근 인기를 얻고 `Elasticsearch`도 추천할만 하지만 `Sunspot`과 함께 한글 검색시 형태소별로 검색이 되는 않는 문제점이 있다. 이것에 대해서 각자 해결해 보도록 하자. 이에 대한 참고문서를 아래에 링크해 두었다.

여기서는 [`AttrSearchable`](https://github.com/mrkamel/attr_searchable)이라는 젬을 사용하여 검새기능을 구현해 보기로 한다.


![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-26_08-09-42_zps3982c776.png)

`1`번 위치에 검색창을 붙이도록 하자.

이를 위해서 우선 `Gemfile`을 열고 아래와 같이 젬을 추가하고,

{%ace edit=true, lang='ruby'%}
gem 'attr_searchable'
```

번들 인스톨한다.

{%ace edit=true, lang='sh'%}
$ bin/bundle install
```

`Post` 모델 파일을 열고 아래와 같이 검색을 위한 모듈을 추가하고 검색할 속성을 지정한다.

{%ace edit=true, lang='ruby'%}
class Post < ActiveRecord::Base
  ...
  include AttrSearchable

  attr_searchable :title, :content
  attr_searchable :comment => "comments.body"

  ...
end
```

`posts#index` 액션에 검색기능을 추가하기 위해서 `app/controllers/posts_controller.rb` 파일을 열고 아래와 같이 변경한다.

{%ace edit=true, lang='ruby'%}
...
def index
  if params[:search]
    @posts = Post.search(params[:search])
  else
    if @category
      @posts = @category.posts.published_posts
    else
      if params[:category_id] == '0'
        @posts = Post.uncategorized_posts
      else
        @posts = Post.published_posts
        @posts = @posts.tagged_with(params[:tag]) if params[:tag]
      end
    end
    @category_name = params[:category_id] == '0' ? "Uncategorized" : (@category ? @category.name : "")
  end
end
...
```

그리고 `app/views/layouts/general.html.erb` 파일을 열고 해당 위치에 아래와 같이 검색 `partial`을 추가한다.

{%ace edit=true, lang='rhtml'%}
...
<div class='medium-3 columns' style="margin-top: 1em">

  <%= render "layouts/search" %>

  <div class='row'>
    <div class='medium-12 columns'>
      <% if user_signed_in? %>
        <p><%= link_to "My Posts <small>( #{Post.myposts(current_user).size} )</small>".html_safe, list_my_posts_path %></p>
      <% end %>
...
```

그리고 `app/views/layouts/_search.html.erb` 파일(`partial`)을 생성하고 아래와 같이 추가한다.

{%ace edit=true, lang='rhtml'%}
<%= form_tag posts_path, :method => :get do %>
<div class="row">
  <div class="large-12 columns">
    <div class="row collapse">
      <div class="small-10 columns">
        <%= text_field_tag :search, params[:search], placeholder: "search" %>
      </div>
      <div class="small-2 columns">
        <%= button_tag icon('magnifying-glass'), class: "button secondary postfix" %>
      </div>
    </div>
  </div>
</div>
<% end %>
```

이제 로컬 서버를 다시 실행하고 브라우저에서 확인한다.

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제10장

---

_**References:**_

1. [Railscasts : Search with Sunspot](http://railscasts.com/episodes/278-search-with-sunspot)
2. [Railscasts : Elasticsearch Part 1](http://railscasts.com/episodes/306-elasticsearch-part-1?language=ko&view=asciicast)
3. [Railscasts : Elasticsearch Part 2](http://railscasts.com/episodes/307-elasticsearch-part-2?language=ko&view=asciicast)
4. [AttrSearchable : Search-engine like fulltext query support for ActiveRecord](https://github.com/mrkamel/attr_searchable)
5. [sunspot 한글 색인 가능하게 하기](https://github.com/sunhohong/planet/wiki/Sunspot-%EC%84%B8%ED%8C%85)
6. [Sunspot과 Lucene을 이용한 풀 텍스트 검색엔진 구축하기](http://thinkreals.wordpress.com/2011/10/10/lucene%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%83%88%EB%A1%9C%EC%9A%B4-%EA%B2%80%EC%83%89%EB%B0%A9%EB%B2%95/)
