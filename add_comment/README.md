# Post에 Comment 붙이기

이 프로젝트에서는 `Post` 모델에 대해서만 댓글기능을 추가하면 되기 때문에 굳이 `Comment` 모델을 `Post` 모델에 대해서 `polymorphic` 관계선언할 필요가 없다.

우선 댓글을 달기 위해서는 로그인상태이어야 한다. 또한 `:user`, `:author`, `:admin` 권한을 가지면 모두 댓글을 달 수 있다. 물론 본인이 작성한 댓글에 대해서만 삭제 권한을 가질 수 있고 댓글은 수정할 수 없도록 하자.

아래는 `CommentAuthorizer` 클래스의 코드를 보여주며, `deletable_by` 인스턴스 메소드를 추가하였다. 삭제할 경우는 권한제한에 무관하게 본인이 작성한 것만 삭제할 수 있으면 된다. 물론 `:admin` 권한을 가진 사용자는 모든 댓글을 삭제할 수 있다.

```ruby
class CommentAuthorizer < ApplicationAuthorizer

  # :user, :author, :admin 권한이 있는 사용자만 댓글을 작성할 수 있음.
  def self.creatable_by?(user)
    user.has_role?(:admin) || user.has_role?(:author) || user.has_role?(:user)
  end

  # :admin 권한이 있거나 본인이 작성한 리소스에 대해서만 삭제할 수 있음.
  def deletable_by?(user)
    resource.user == user || user.has_role?(:admin)
  end

end
```

이미 앞서 `Comment` 모델을 작성하였기 때문에 `:body` 속성을 필수항목으로 지정한다.

```ruby
class Comment < ActiveRecord::Base
  resourcify
  include Authority::Abilities

  belongs_to :user
  belongs_to :post

  validates :body, presence: true

end
```

이제 `app/controllers/comments_controller.rb` 파일을 열고 `create`와 `destroy` 액션을 작성하자.

```ruby
class CommentsController < ApplicationController
  before_action :authenticate_user!
  before_action :set_post
  before_action :set_comment, only: :destroy

  def create
    authorize_action_for Comment
    @comment = @post.comments.new(comment_params)
    @comment.user = current_user
    @comment.save
    respond_to do | format |
      format.js
    end
  end

  def destroy
    authorize_action_for @comment
    @comment.destroy
    respond_to do | format |
      format.js
    end
  end

  private

  def set_post
    @post = Post.find(params[:post_id])
  end

  def set_comment
    @comment = @post.comments.find(params[:id])
  end

  def comment_params
    params.require(:comment).permit(:body)
  end

end
```

여기서는 `ajax`로 댓글을 생성하고 삭제하도록 하자. 이렇게 하는 이유는 `post` 뷰 페이지에서 페이지의 이동없이 바로 댓글을 추가하고 삭제하기 위해서이다.

`app/views/posts/_post.html.erb` 파일을 열고 댓글의 갯수만 표시하도록 하자. 이것은 `posts#index` 액션의 뷰 페이지에서 보여줄 `partial` 템플릿 파일이다.

```html
...
<% if post.comments.count > 0 %>
  <div class='comments'>
    <%= icon('comment') %> <%= post.comments.count %>
  </div>
<% end %>
...
```

`app/views/posts/show.html.erb` 파일을 열고 하단에 아래와 같이 댓글 표시할 부분으로 `partial` 뷰 템플릿을 렌더링하도록 작성한다.
```html
<%= render partial: 'comments/comments', locals: { post: @post } %>
```

이제 `app/views/comments/_comments.html.erb` 파일을 생성하고 아래와 같이 작성한다.

```html
<% comment = post.comments.new %>
<div id="comments_<%=post.id%>" class='post_comments'>
  <h4>Comments <small>( <%= post.comments.count %> )</small></h4>
  <div id="list_of_comments_<%=post.id%>" class='list_of_comments'>
    <ul class='comments'>
      <%= render post.comments %>
    </ul>
  </div>
  <div id="form_of_comments_<%=post.id%>" class="comment_form">
    <%= render partial: "comments/form", locals: { comment: comment } %>
  </div>
</div>
```

실제로 댓글을 표시할 `partial` 템플릿 파일을 작성하기 위해서 `app/views/comments/_comment.html.erb` 파일을 생성하고 아래와 같이 작성한다.

```html
<% if comment.body.present? %>
<li id="comment_<%=comment.id%>" class="comment">
  <%= simple_format comment.body %>
  <div style='text-align:right;margin-top:0;border-top:1px solid #eaeaea;'>
    <small>- <%= comment.user.email %>, <%= time_ago_in_words(comment.created_at)%> ago</small>
    <% if user_signed_in? && current_user.can_delete?(comment) %>
      <%= link_to icon('trash'), [comment.post, comment], method: :delete, remote: true, data: { confirm: 'Are you sure?'} %>
    <% end %>
  </div>
</li>
<% end %>
```

여기시 주목할 것은 하단에 있는 댓글 삭제 링크다. `link_to` 메소드의 옵션으로 `remote: true`를 지정했다. 이로써 삭제 링크를 클릭했을 때 `HTTP DELETE` 메소드를 이용하여 `posts/:post_id/comments/:id` URI로 `ajax` 요청을 하게 된다. 결과적으로 `comments#destroy` 액션이 동작하게 되고 `destroy.js.erb`을 렌더링하여 응답하게 된다.

위에서 댓글 입력을 위해서 사용하게 되는 `Comment` 모델의 폼 `partial` 파일을 작성하자. `app/views/comments/_form.html.erb` 파일을 생성하고 아래와 같이 작성한다.

```html
<%= simple_form_for [comment.post, comment], remote: true do | f | %>
  <%= f.input :body, placeholder: 'Add a comment', label: false, input_html: { class: "form-control", rows: 5 } %>
  <div style='text-align:right;'>
  <%= f.button :submit, class: 'button tiny radius' %>
  </div>
<% end %>
```

여기서는 주목할 것은 `simple_form_for` 헬퍼 메소드의 옵션으로 `remote: true`를 지정했다는 것이다. 이로써 `HTTP POST` 메소드를 이용하여 `/posts/:post_id/comments` URI를 `ajax`로 요청하게 되고 서버에서는 `comments#create` 액션이 호출된 후 결과로써 `create.js.erb`가 렌더링된 후 사용자의 브라우저로 보내지게 된다.

이상의 것을 정리하면, 댓글 생성할 때 `create` 액션 결과로  `app/views/comments/create.js.erb` 파일이 렌더링되어 클라이언트로 응답하고,

```javascript
<% if @comment.errors.size == 0 %>
  $('<%=j render @comment %>').appendTo($("#list_of_comments_<%=@post.id %> ul")).hide().fadeIn('fast');
  $("#form_of_comments_<%=@post.id %> form")[0].reset();
<% else %>
  alert("Please submit after commenting...");
<% end %>
```

댓글 삭제시는 `destroy` 액션의 응답으로 `app/views/comments/destroy.js.erb` 파일을 렌더링하여 클라이언트로 응답하게 된다.

```javascript
$("#comment_<%=@comment.id %>").slideUp('fast');
```

댓글 관련 `CSS`를 정리하여  `app/assets/stylesheets/comments.css.scss` 파일로 아래와 같이  추가하고,

```css
ul.comments {
  li.comment {
    list-style: none;
    border-left:1em solid #eaeaea;
    padding-left:.5em;
    margin-bottom: 1em;
    p {
      margin-bottom: .3em;
    }
  }
}

.post_comments {
  margin-bottom:2em;
}
```

이 파일을 `application.scss` 파일에 임포트한다.

```css
...
@import 'comments';
...
```

`_post.html.erb` 파일에서 사용할 `CSS`를 수정하기 위해 `app/assets/posts/posts.css.scss` 파일을 열고 아래와 같이 수정한다.

```css
...
.comments {
  font-size: .8em;
}
.tags {
  margin-top: .3em;
}
.actions { margin-top:1em; }
...
```

`app/views/posts/show.html.erb` 파일을 열고 약간의 아래와 같이 약간의 수정을 한다.

```html
<div class='post'>
  <div class='title'>
    <H3><%= @post.title %></H3>
  </div>
  <div class='author'>
    created by <%= @post.user.email %>, <%= time_ago_in_words(@post.created_at) %> ago
  </div>
  <div class='content'>
    <%= simple_format @post.content %>
  </div>
  <div class='category'>
     <%= icon('folder') + ' ' + @post.category.name %>
  </div>
  <div class='tags'>
     <%= icon_tags(@post.tag_list) %>
  </div>
  <% if user_signed_in? && current_user.can_update?(@post) %>
    <div class='published'>
      <%= published_icon @post.published %>
    </div>
  <% end %>
</div>
<hr>

<div style='text-align:right;'>
  <%= link_to 'Edit', edit_post_path(@post), class: 'button tiny radius' %>
  <%= link_to 'Back', posts_path, class: 'button tiny radius' %>
</div>

<%= render partial: 'comments/comments', locals: { post: @post } %>
```

이상에서 작업을 내용을 요약하면, `posts#index` 액션으로 뷰 페이지를 볼 때는 댓글의 갯수만 보이도록 하고 `posts#show` 액션으로 댓글을 하나씩 볼 때 댓글의 상세한 내용을 볼 수 있게 하였다.

자 이제, 마직막으로 검색 기능을 추가해 보자.

---

> **소스보기** https://github.com/LuciusChoi/foundblog/tree/제9장
