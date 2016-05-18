# Category/Post/Comment 모델의 생성

글을 분류하기 위한 `Category` 리소스를 생성하고, 이후에 생성할 `Post` 모델과는 `has_many`, `belongs_to` 메소드로 관계선언한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/rails g scaffold Category user:references name
Running via Spring preloader in process 97019
      invoke  active_record
      create    db/migrate/20160513085505_create_categories.rb
      create    app/models/category.rb
      invoke    test_unit
      create      test/models/category_test.rb
      create      test/fixtures/categories.yml
      invoke  resource_route
       route    resources :categories
      invoke  scaffold_controller
      create    app/controllers/categories_controller.rb
      invoke    erb
      create      app/views/categories
      create      app/views/categories/index.html.erb
      create      app/views/categories/edit.html.erb
      create      app/views/categories/show.html.erb
      create      app/views/categories/new.html.erb
      create      app/views/categories/_form.html.erb
      invoke    test_unit
      create      test/controllers/categories_controller_test.rb
      invoke    helper
      create      app/helpers/categories_helper.rb
      invoke      test_unit
      invoke    jbuilder
      create      app/views/categories/index.json.jbuilder
      create      app/views/categories/show.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/categories.coffee
      invoke    scss
      create      app/assets/stylesheets/categories.scss
      invoke  scss
   identical    app/assets/stylesheets/scaffolds.scss
{%endace%}

그리고 `db/migrate/20160513085505_create_categories.rb` 파일을 열고 `:name` 속성에 `null: false` 옵션을 추가하여 필수항목으로 지정한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class CreateCategories < ActiveRecord::Migration
  def change
    create_table :categories do |t|
      t.references :user, index: true
      t.string :name, null: false

      t.timestamps
    end
  end
end
{%endace%}


이제 블로그의 게시물 내용을 저장할 `Post` 리소스를 생성한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/rails g scaffold Post category:references user:references title content:text published:boolean
Running via Spring preloader in process 96447
      invoke  active_record
      create    db/migrate/20160513085038_create_posts.rb
      create    app/models/post.rb
      invoke    test_unit
      create      test/models/post_test.rb
      create      test/fixtures/posts.yml
      invoke  resource_route
       route    resources :posts
      invoke  scaffold_controller
      create    app/controllers/posts_controller.rb
      invoke    erb
      create      app/views/posts
      create      app/views/posts/index.html.erb
      create      app/views/posts/edit.html.erb
      create      app/views/posts/show.html.erb
      create      app/views/posts/new.html.erb
      create      app/views/posts/_form.html.erb
      invoke    test_unit
      create      test/controllers/posts_controller_test.rb
      invoke    helper
      create      app/helpers/posts_helper.rb
      invoke      test_unit
      invoke    jbuilder
      create      app/views/posts/index.json.jbuilder
      create      app/views/posts/show.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/posts.coffee
      invoke    scss
      create      app/assets/stylesheets/posts.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.scss
{%endace%}

여기서 `Post` 모델은 카테고리별로 분류할 수 있어야 한다. 따라서 `category:references` 옵션을 추가한다. `published:boolean`은 작성한 글을 다른 사람들이 보지 못하게 할 목적으로 추가한다. 그리고 디폴트 값은 `false`로 지정하고, `title`과 `content` 속성은 `null: false`로 옵션을 추가하여 필수항목으로 지정하기 위해서 `db/migrate/20160513085038_create_posts.rb` 파일을 열어서 아래와 같이 변경한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.references :category, index: true, foreign_key: true
      t.references :user, index: true, foreign_key: true
      t.string :title, null: false
      t.text :content, null: false
      t.boolean :published, default: false

      t.timestamps null: false
    end
  end
end
{%endace%}

다음으로, 글에 대한 댓글을 달기 위해서 `Comment` 리소스를 생성한다. 이때는 `resource` 제너레이터를 사용한다. `scaffold` 제너레이터와의 차이점은 뷰 관련 리소스(뷰 템플릿 파일과 관련 레이아웃과 css 파일)을 생성하지 않는다는 것이다. 이에 대한 자세한 내용은 [`여기`](http://www.question-defense.com/2009/12/29/rails-resource-vs-rails-scaffold)를 참고하기 바란다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/rails g resource Comment user:references post:references body:text
Running via Spring preloader in process 96900
      invoke  active_record
      create    db/migrate/20160513085339_create_comments.rb
      create    app/models/comment.rb
      invoke    test_unit
      create      test/models/comment_test.rb
      create      test/fixtures/comments.yml
      invoke  controller
      create    app/controllers/comments_controller.rb
      invoke    erb
      create      app/views/comments
      invoke    test_unit
      create      test/controllers/comments_controller_test.rb
      invoke    helper
      create      app/helpers/comments_helper.rb
      invoke      test_unit
      invoke    assets
      invoke      coffee
      create        app/assets/javascripts/comments.coffee
      invoke      scss
      create        app/assets/stylesheets/comments.scss
      invoke  resource_route
       route    resources :comments
{%endace%}

마이그레이션 파일(db/migrate/20160513085339_create_comments.rb)을 열고 `body` 속성은 필수항목으로 지정한다. 

{%ace edit=false, lang='ruby', theme='monokai'%}
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.references :user, index: true, foreign_key: true
      t.references :post, index: true, foreign_key: true
      t.text :body, null: false

      t.timestamps null: false
    end
  end
end
{%endace%}

`Post` 모델 클래스에서 `has_many :comments, dependent: :destroy`를 추가한다. 그러나 `User` 모델 클래스에서 `has_many :comments, dependent: :destroy` 관계선언은 생략해도 된다. 왜냐하면 `Post` 모델 객체가 삭제될 때 이미 `Comment` 객체들도 삭제될 것이기 때문이다.

이제 지금까지 생성한 모델에서 대해서 마이그레이션 작업을 수행한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/rake db:migrate
== 20140609052247 CreatePosts: migrating ======================================
-- create_table(:posts)
   -> 0.0067s
== 20140609052247 CreatePosts: migrated (0.0068s) =============================

== 20140609053323 CreateCategories: migrating =================================
-- create_table(:categories)
   -> 0.0011s
== 20140609053323 CreateCategories: migrated (0.0012s) ========================
{%endace%}

`Category` 모델 클래스 파일을 열고 아래와 같이 `has_many` 메소드를 추가한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class Category < ActiveRecord::Base
  belongs_to :user
  has_many :posts, dependent: :nullify
end
{%endace%}

`User` 모델 클래스 파일을 열고 아래와 같이 `has_many` 메소드를 추가한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
has_many :categories, dependent: :nullify
has_many :posts, dependent: :destroy
{%endace%}

이 메소드에서 사용한 `:dependent` 옵션에 `:nullify` 값을 지정하면, 부모 클래스 객체가 삭제될 때 자식 클래스 모델도 함께 삭제(`:destroy`)하지 않고 단지 자식 모델의 `foreign key`를 `null` 값으로 지정하게 된다.

다음은 로그인이 필요한 컨트롤러의 액션을 생각해 보자.
`posts` 컨트롤러의 모든 액션에 대해서 인증이 필요하다고 선언한 후, 예외적인 상황을 고려하는 것이 적합한 로직(black-list 방식)이 된다. 만약, 새로운 액션을 하나 추가만 했을 경우에는 기본적으로는 인증을 필요로 하는 상태가 된다. 따라서 나중에 이 액션이 인증이 필요없다고 판단할 때 예외적인 상황으로 등록하면 되는 것이다.

`posts_controller.rb` 파일의 상단에 아래와 같이 추가한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class PostsController < ApplicationController
  before_action :authenticate_user!, except: [ :index, :show ]
...
{%endace%}

여기서 사용한 `authentiate_user!` 메소드는 `User` 모델을 `devise` 제너레이터로 생성할 때 디폴트로 제공되는 인증 메소드다.




`Post` 리소스에 대한 권한 로직을 구현하기 위해서 `app/authorizers/post_authorizer.rb` 파일을 생성하고 아래와 같이 작성한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class PostAuthorizer < ApplicationAuthorizer

  # :author, :admin 권한이 있는 사용자만 글을 작성할 수 있음.
  def self.creatable_by?(user)
    user.has_role?(:author) || user.has_role?(:admin)
  end

end
{%endace%}

`Comment` 리소스에 대한 권한 로직을 구현하기 위해서 `app/authorizers/comment_authorizer.rb` 파일을 생성하고 아래와 같이 작성한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class CommentAuthorizer < ApplicationAuthorizer

  # :user, :author, :admin 권한이 있는 사용자만 댓글을 작성할 수 있음.
  def self.creatable_by?(user)
    user.has_role?(:admin) || user.has_role?(:author) || user.has_role?(:user)
  end

end
{%endace%}

`Category` 리소스에 대한 권한 로직을 구현하기 위해서 `app/authorizers/category_authroizer.rb` 파일을 생성하고 아래와 같이 작성한다. `Category` 모델은 `admin` 권한이 있는 경우에만 객체를 생성/수정/삭제할 수 있도록 한다. 그러나 이미 디폴트 상태에서 `admin` 권한만이 모든 모델에 대한 권한을 가지기 때문에 별도로 권한을 지정할 필요는 없다. 따라서 `CategoryAuthroizer` 클래스는 작성하지 않아도 된다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class CategoryAuthorizer < ApplicationAuthorizer

  # :admin 권한이 있는 사용자만 카테고리를 작성/수정/삭제할 수 있음.
  # def self.default(adjective, user)
  #  user.has_role?(:admin)
  # end

end
{%endace%}

이와 관련하여 `User` 모델에는 `Authority::UserAbilities` 모듈을 인클루드해야 하고, 나머지 권한설정을 적용하려는 모든 모델에는 `Authority::Abilities` 모듈을 인클루드해야 한다. 그리고 `rolify` 젬의 용도에 따라 `User` 모델에는 `rolify` 매크로를 지정해 주어야 하고 나머지 `Role`을 적용할 모든 모델에는 `resourcify` 매크로를 지정해 주어야 한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class User < ActiveRecord::Base
  rolify
  include Authority::UserAbilities
  ...
end

class Category < ActiveRecord::Base
  resourcify
  include Authority::Abilities
  ...
end

class Post < ActiveRecord::Base
  resourcify
  include Authority::Abilities
  ...
end

class Comment < ActiveRecord::Base
  resourcify
  include Authority::Abilities
  ...
end
{%endace%}

이제는 컨트롤러 내에서 권한체크를 하는 방법에 대해서 알아 보자.

먼저, `posts` 컨트롤러의 액션들에 대해서 권한 설정을 구체적으로 해 보자.

이미 `posts` 컨트롤러에 대해서 사용자 인증을 위해  `authenticate_user! except: [ :index, :show ]`와 같이 선언한 바 있다. 나머지 액션들, 즉, `:new`, `:create`, `:edit`, `:update`, `:destroy` 액션들은 반드시 인증이 된 상태(로그인 상태)에서만 접근할 수 있게 되는데, 위에서 가정한 권한로직에 따라 `:edit`, `:update`, `:destroy` 액션에 대해서는 본인이 작성한 글에 대해서만 권한을 가져야 하기 때문에, `Authoriy` 젬에서 컨트롤러에 대해서 제공하는 `authorize_action_for` 메소드를 아래와 같이 이들 액션에 대해서 추가한다. 그리고 `:new`와 `:create` 액션에 대해서는 `Post` 클래스에 대한 권한 체크를 해야 하므로 `posts` 컨트롤러 클래스에 선언해 준다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class PostsController < ApplicationController
  ...
  authorize_actions_for Post, only: [ :new, :create ]

  def edit
    authorize_action_for @post
  end

  def update
    authorize_action_for @post
    ...
  end

  def destroy
    authorize_action_for @post
    ...
  end
  ...

end
{%endace%}

따라서 현재 로그인한 사용자가 이들 액션에 접근할 때 `authorize_action_for @post`는 `PostAuthorizer` 클래스에서 해당 액션과 연결되는 권한체크 메소드를 호출하게 되는 것이다. 이와 같이 액션과 연결되는 권한체크 메소들의 매핑 정보는 `config/initializers/authority.rb` 파일에 디폴트로 정의되어 있다.

{%ace edit=false, lang='ruby', theme='monokai'%}
# Defaults are as follows:
#
# config.controller_action_map = {
#   :index   => 'read',
#   :show    => 'read',
#   :new     => 'create',
#   :create  => 'create',
#   :edit    => 'update',
#   :update  => 'update',
#   :destroy => 'delete'
# }
{%endace%}

이와 같은 `controller_action_map`의 정의에 따라 `current_user`는 `can_read?`, `can_create?`, `can_update?`, `can_delete?`와 같은 동사형 권한체크 메소드를 사용할 수 있게 된다

{%ace edit=false, lang='ruby', theme='monokai'%}
# ABILITIES
# =========
# Teach Authority how to understand the verbs and adjectives in your system. Perhaps you
# need {:microwave => 'microwavable'}. I'm not saying you do, of course. Stop looking at
# me like that.
#
# Defaults are as follows:
#
# config.abilities =  {
#   :create => 'creatable',
#   :read   => 'readable',
#   :update => 'updatable',
#   :delete => 'deletable'
# }
{%endace%}

또한 `abilities`의 정의에 따라 동사와 형용사형의 메소드를 연결시키도록 해 준다.

뷰 템블릿에서 이러한 동사형 또는 형용사형의 권한체크 메소드를 `if` 조건절에서 사용하면 `boolean` 값을 반환한다. 그러나 컨트롤러에서 `authorize_action_for` 메소드를 사용할 때 권한이 없는 경우 `403, Security Error` 예외를 발생키시고 `publinc/403.html` 웹페이지를 보여주게 된다. 그러나 이러한 접근에러 페이지 대신에 `flash` 메시지로 보여주면 한결 전체적인 흐름이 부드러워 질 수 있다. 이를 위해서 `application_controller.rb` 파일에 아래와 같이 추가한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class ApplcationController < ActionController::Base
  ...

  def authority_forbidden(error)
    Authority.logger.warn(error.message)
    redirect_to request.referrer.presence || root_path, :alert => 'You are not authorized to complete that action.'
  end

  ...

end
{%endace%}

아직 보안위반에 대한 `flash` 메시지를 제대로 볼 수 없다. 나중에 `flash` 메시지를 보이기 위한 헬퍼 메소드를 작성할 때 함께 다루도록 하겠다.

지금까지 작업한 내용을 로컬 저장소로 커밋한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git add .
$ git commit -m "제03장 : Category/Post/Comment 모델의 생성"
$ git tag "제03장"
{%endace%}


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제03장
