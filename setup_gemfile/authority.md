# authority 젬

[`authority`](https://github.com/nathanl/authority) 젬은 특정 리소스에 대한 모델과 컨트롤러 액션들에 대한 접근권한을 제한하기 위해 사용한다.

이 젬을 인스톨하기 위해서 아래와 같이 명령을 실행한다.

```bash
$ bin/rails g authority:install
Running via Spring preloader in process 98936
      create  app/authorizers
      create  app/authorizers/application_authorizer.rb
      create  config/initializers/authority.rb
      create  public/403.html

Install complete! See the README on Github for instructions 
on getting your app running with Authority.
```

생성된 파일 중 `app/authorizers` 디렉토리에 있는 `application_authroizer.rb` 파일에는 `ApplicationAuthorizer` 클래스가 있는데, 이것은 어플리케이션 전체에 대해서 영향을 미치는 최상위 레벨의 권한을 정의하는 곳이다.

```ruby
class ApplicationAuthorizer < Authority::Authorizer
  def self.default(adjective, user)
    false
  end
end
```

`authority` 젬을 사용할 때, 어플리케이션에서의 최종 권한 메소드는 `ApplicationAuthorizer` 클래스의 `default` 클래스 메소드다. 이 메소드의 반환값은 항상 `false`다. 즉, 이와 같은 디폴트 상태에서는 어플리케이션 내의 모든 모델에 대해서 권한 체크 메소드(`updateable_by` 또는 `can_update?`)를 호출할 때 항상 `false` 값을 반환하여 접근권한이 없게 되고, 또한, 모델과 연결되는 컨트롤러의 모든 액션들에 대해서 `authorize_action_for`와 같은 권한체크 메소드를 호출하게 될 때 역시 `false` 값을 반환하여 접근권한이 없게 된다.

언급했던 바와 같이 모델 클래스나 모델 클래스 객체에 대해서 권한체크를 할 때 뷰 파일내에서는 동사형과 형용사형의 메소드를 사용할 수 있다. 예를 들어 모델 클래스나 객체에 대해서 `update` 메소드에 대해서 권한을 물을 때 동사형인 `can_update?` 메소드나 형용사형인 `updatable_by?` 메소드 중에 아무거나 사용해도 된다. 단, 동사형의 메소드를 사용할 때는 인증용 모델인 `User` 클래스내에 `include Authority:UserAbilities`를 추가하고 형용사형의 메소드를 사용할 때는 대상 모델들의 클래스내에 `include Authority::Abilities`를 추가해야 한다.

특정 모델에 대해서 별도의 권한 로직은 구현하고자 할 때는, 모델이름과 `Authorizer`를 합성한 클래스 파일, 즉, 예를 들어 `Post` 모델에 대해서는 `PostAuthorizer` 클래스 파일에 정의하면 된다. 이 `Authorizer` 클래스 파일들은 `app/authorizers` 디렉토리에 위치한다.

만약 해당 모델에 대한 `Authorizer` 클래스 파일이 존재하지 않으면 최상위 `Authorizer` 클래스인 `ApplicationAuthorizer` 클래스에서 권한체크를 하게 된다.

현재 상태에서는 각 모델에 대한 `Authorizer` 클래스가 없기 때문에 모든 권한로직은 `ApplicationAuthorizer` 클래스에 집중되어 있다. 그러나 나중에 각각의 모델별로 권한로직을 분산할 수 있고 여러개의 모델이 동일한 `Authorizer` 클래스를 공유할 수 있도록 로직을 구성할 수 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/authority_strategy_zpsa6415ee4.png)

더 자세한 내용은 [`사용자 권한 설정하기`](http://luciuschoi.gitbooks.io/exploring_devise/user_authorization/README.html)를 참고하기 바란다.

자, 이제 실제로 권한 로직을 구현해 보자.

여기서 구현할 권한 로직은 간단하다.

로그인하지 않은 상태에서 사용자는 `FoundBlog`의 모든 게시글을 읽을 수만 있다(`:read`). 로그인한 상태에서 `:user` 권한을 가진 사용자는 댓글을 작성할 수 있고, 관리자로부터 `:author` 권한을 부여 받은 사용자는 새글을 작성할 수 있을 뿐만 아니라 본인이 작성한 글 또한  수정(`:update`)하거나 삭제(`:destroy`)할 수 있도록 하자. 물론 관리자 권한을 가진 사용자는 모든 게시물에 대해서 작성, 수정 및 삭제 권한을 가진다.

위의 권한 로직을 코드로 구현하면 아래와 같다.

최상위 권한 로직을 구현하기 위해서 `app/authorizers/application_authorizer.rb` 파일을 열고 아래와 같이 변경한다.

```ruby
class ApplicationAuthorizer < Authority::Authorizer

  # :admin 권한이 있는 사용자는 모든 권한을 가짐.
  def self.default(adjective, user)
    user.has_role? :admin
  end

  # :admin 권한이 있거나 :author 권한을 가진 경우 본인이 작성한 리소스만을 수정할 수 있음.
  def updatable_by?(user)
    (user.has_role?(:author) && resource.user == user) || user.has_role?(:admin)
  end

  # :admin 권한이 있거나 :author 권한을 가진 경우 본인이 작성한 리소스만을 삭제할 수 있음.
  def deletable_by?(user)
    (user.has_role?(:author) && resource.user == user) || user.has_role?(:admin)
  end

end
```

`Post` 리소스에 대한 권한 로직을 구현하기 위해서 `app/authorizers/post_authorizer.rb` 파일을 생성하고 아래와 같이 작성한다.

```ruby
class PostAuthorizer < ApplicationAuthorizer

  # :author, :admin 권한이 있는 사용자만 글을 작성할 수 있음.
  def self.creatable_by?(user)
    user.has_role?(:author) || user.has_role?(:admin)
  end

end
```

`Comment` 리소스에 대한 권한 로직을 구현하기 위해서 `app/authorizers/comment_authorizer.rb` 파일을 생성하고 아래와 같이 작성한다. (`Comment` 리소스는 아직 작성하지 않은 상태)

```ruby
class CommentAuthorizer < ApplicationAuthorizer

  # :user, :author, :admin 권한이 있는 사용자만 댓글을 작성할 수 있음.
  def self.creatable_by?(user)
    user.has_role?(:admin) || user.has_role?(:author) || user.has_role?(:user)
  end

end
```

`Category` 리소스에 대한 권한 로직을 구현하기 위해서 `app/authorizers/category_authroizer.rb` 파일을 생성하고 아래와 같이 작성한다.

```ruby
class CategoryAuthorizer < ApplicationAuthorizer

  # :admin 권한이 있는 사용자만 카테고리를 작성/수정/삭제할 수 있음.
  def self.default(adjective, user)
    user.has_role?(:admin)
  end

end
```

이와 관련하여 `User` 모델에는 `Authority::UserAbilities` 모듈을 인클루드해야 하고, 나머지 권한설정을 적용하려는 모든 모델에는 `Authority::Abilities` 모듈을 인클루드해야 한다. 그리고 `rolify` 젬의 용도에 따라 `User` 모델에는 `rolify` 매크로를 지정해 주어야 하고 나머지 `Role`을 적용할 모든 모델에는 `resourcify` 매크로를 지정해 주어야 한다.

```ruby
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
```

이제는 컨트롤러 내에서 권한체크를 하는 방법에 대해서 알아 보자.

먼저, `posts` 컨트롤러의 액션들에 대해서 권한 설정을 구체적으로 해 보자.

이미 `posts` 컨트롤러에 대해서 사용자 인증을 위해  `authenticate_user! except: [ :index, :show ]`와 같이 선언한 바 있다. 나머지 액션들, 즉, `:new`, `:create`, `:edit`, `:update`, `:destroy` 액션들은 반드시 인증이 된 상태(로그인 상태)에서만 접근할 수 있게 되는데, 위에서 가정한 권한로직에 따라 `:edit`, `:update`, `:destroy` 액션에 대해서는 본인이 작성한 글에 대해서만 권한을 가져야 하기 때문에, `Authoriy` 젬에서 컨트롤러에 대해서 제공하는 `authorize_action_for` 메소드를 아래와 같이 이들 액션에 대해서 추가한다. 그리고 `:new`와 `:create` 액션에 대해서는 `Post` 클래스에 대한 권한 체크를 해야 하므로 `posts` 컨트롤러 클래스에 선언해 준다.

```ruby
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
```

따라서 현재 로그인한 사용자가 이들 액션에 접근할 때 `authorize_action_for @post`는 `PostAuthorizer` 클래스에서 해당 액션과 연결되는 권한체크 메소드를 호출하게 되는 것이다. 이와 같이 액션과 연결되는 권한체크 메소들의 매핑 정보는 `config/initializers/authority.rb` 파일에 디폴트로 정의되어 있다.

```ruby
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
```

이와 같은 `controller_action_map`의 정의에 따라 `current_user`는 `can_read?`, `can_create?`, `can_update?`, `can_delete?`와 같은 동사형 권한체크 메소드를 사용할 수 있게 된다

```ruby
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
```

또한 `abilities`의 정의에 따라 동사와 형용사형의 메소드를 연결시키도록 해 준다.

뷰 템블릿에서 이러한 동사형 또는 형용사형의 권한체크 메소드를 `if` 조건절에서 사용하면 `boolean` 값을 반환한다. 그러나 컨트롤러에서 `authorize_action_for` 메소드를 사용할 때 권한이 없는 경우 `403, Security Error` 예외를 발생키시고 `publinc/403.html` 웹페이지를 보여주게 된다. 그러나 이러한 접근에러 페이지 대신에 `flash` 메시지로 보여주면 한결 전체적인 흐름이 부드러워 질 수 있다. 이를 위해서 `application_controller.rb` 파일에 아래와 같이 추가한다.

```ruby
class ApplcationController < ActionController::Base
  ...

  def authority_forbidden(error)
    Authority.logger.warn(error.message)
    redirect_to request.referrer.presence || root_path, :alert => 'You are not authorized to complete that action.'
  end

  ...

end
```

아직 보안위반에 대한 `flash` 메시지를 제대로 볼 수 없다. 나중에 `flash` 메시지를 보이기 위한 헬퍼 메소드를 작성할 때 함께 다루도록 하겠다.

---

> **소스보기** https://github.com/LuciusChoi/foundblog/tree/제2.6장

---

_**References:**_

1. [nathanl/authority](https://github.com/nathanl/authority)
2. [Devise젬 파헤치기 - 사용자 권한 설정하기](http://luciuschoi.gitbooks.io/exploring_devise/user_authorization/README.html)
