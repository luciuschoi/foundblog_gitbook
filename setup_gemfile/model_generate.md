# Category/Post/Comment 모델의 생성

먼저 `Category` 리소스를 생성하고 다음에 생성할 `Post` 모델과는 `has_many`, `belongs_to` 메소드로 관계선언을 할 것이다.

```bash
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
```

그리고 `db/migrate/20160513085505_create_categories.rb` 파일을 열고 `:name` 속성에 `null: false` 옵션을 추가하여 필수항목으로 지정한다.

```ruby
class CreateCategories < ActiveRecord::Migration
  def change
    create_table :categories do |t|
      t.references :user, index: true
      t.string :name, null: false

      t.timestamps
    end
  end
end
```

{%ace edit=true, lang='ruby'%}
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


이제 블로그의 게시물 내용을 저장할 `Post` 리소스를 만들도록 하자.

```bash
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
```

여기서 `Post` 모델은 카테고리별로 분류할 수 있어야 한다. 따라서 `category:referenes` 파라미터를 추가했다. `published:boolean`은 작성한 글을 다른 사람들이 보지 못하게 할 목적으로 추가했다. 그리고 디폴트 값은 `false`로 지정하고, `title`과 `content` 속성은 `null: false`로 옵션을 추가하여 필수항목으로 지정하기 위해서 `db/migrate/20160513085038_create_posts.rb` 파일을 열어서 아래와 같이 변경한다.

```ruby
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
```

다음으로, 글에 대한 댓글을 달기 위해서 `Comment` 리소스를 생성한다. 이때는 `resource` 제너레이터를 사용한다. `scaffold` 제너레이터와의 차이점은 뷰 템플릿 파일과 관련 레이아웃과 css 파일을 생성하지 않는다는 것이다. 이에 대한 자세한 내용은 [`여기`](http://www.question-defense.com/2009/12/29/rails-resource-vs-rails-scaffold)를 참고하기 바란다.

```bash
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
```

`Post` 모델 클래스에서 `has_many :comments, dependent: :destroy`를 추가한다. 그러나 `User` 모델 클래스에서 `has_many :comments, dependent: :destroy` 관계선언은 생략해도 된다. 왜냐하면 `Post` 모델 객체가 삭제될 때 이미 `Comment` 객체들도 삭제될 것이기 때문이다.

이제 지금까지 생성한 모델에서 대해서 마이그레이션 작업을 수행한다.

```bash
$ bin/rake db:migrate
== 20140609052247 CreatePosts: migrating ======================================
-- create_table(:posts)
   -> 0.0067s
== 20140609052247 CreatePosts: migrated (0.0068s) =============================

== 20140609053323 CreateCategories: migrating =================================
-- create_table(:categories)
   -> 0.0011s
== 20140609053323 CreateCategories: migrated (0.0012s) ========================
```

`Categroy` 모델 클래스 파일을 열고 아래와 같이 `has_many` 메소드를 추가해 준다.

```ruby
class Category < ActiveRecord::Base
  belongs_to :user
  has_many :posts, dependent: :nullify
end
```

`User` 모델 클래스 파일을 열고 아래와 같이 `has_many` 메소드를 추가해 준다.

```ruby
has_many :categories, dependent: :nullify
has_many :posts, dependent: :destroy
```

이 메소드에서 사용한 `:dependent` 옵션으로 `:nullify` 값을 지정하면, 부모 클래스 객체가 삭제될 때 자식 클래스 모델도 함께 삭제(`:destroy`)하지 않고 단지 자식 모델의 `foreign key`를 `null` 값으로 지정하게 된다.

다음은 로그인이 필요한 컨트롤러의 액션을 생각해 보자.
`posts` 컨트롤러의 모든 액션에 대해서는 우선 인증이 필요하다고 선언하고 예외적인 상황을 고려하는 것이 이 컨텍스 상에서 적합한 로직(black-list 방식)이 된다. 만약, 새로운 액션을 하나 추가만 했을 경우 이 상황에서는 기본적으로는 인증이 필요한 상태로 된다. 따라서 나중에 이 액션이 인증이 필요없다고 판단할 때 예외적인 상황으로 등록하면 되는 것이다.

`posts_controller.rb` 파일의 상단에 아래와 같이 추가한다.

```ruby
class PostsController < ApplicationController
  before_action :authenticate_user!, except: [ :index, :show ]
...
```

여기서 사용한 `authentiate_user!` 메소드는 `User` 모델을 `devise` 제너레이터로 생성하면 디폴트로 제공되는 인증 메소드이다.

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02.5장
