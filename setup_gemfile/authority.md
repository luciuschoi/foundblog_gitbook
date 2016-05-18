# authority 젬

[`authority`](https://github.com/nathanl/authority) 젬은 특정 리소스에 대한 모델과 컨트롤러 액션들에 대한 접근권한을 제한하기 위해 사용한다.

처음부터 작업을 따라 해온 경우라면 이미 `Gemfile` 파일에 아래와 같이 젬을 추가한 후 번들 인스톨한 상태다.

{%ace edit=false, lang='ruby', theme='monokai'%}
gem "authority"
{%endace%}

이 젬을 셋업하기 위해서 아래와 같이 명령을 실행한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ bin/rails g authority:install
Running via Spring preloader in process 98936
      create  app/authorizers
      create  app/authorizers/application_authorizer.rb
      create  config/initializers/authority.rb
      create  public/403.html

Install complete! See the README on Github for instructions
on getting your app running with Authority.
{%endace%}

생성된 파일 중 `app/authorizers` 디렉토리에 있는 `application_authroizer.rb` 파일에는 `ApplicationAuthorizer` 클래스가 있는데, 이것은 애플리케이션 전체에 대해서 영향을 미치는 최상위 레벨의 권한을 정의하는 곳이다.

{%ace edit=false, lang='ruby', theme='monokai'%}
class ApplicationAuthorizer < Authority::Authorizer
  def self.default(adjective, user)
    false
  end
end
{%endace%}

`authority` 젬을 사용할 때, 애플리케이션에서의 최종 권한 메소드는 `ApplicationAuthorizer` 클래스의 `default` 클래스 메소드다. 이 메소드의 반환값은 항상 `false`다. 즉, 이와 같이 디폴트 상태에서는 애플리케이션 내의 모든 모델에 대해서 권한 체크 메소드(`updateable_by` 또는 `can_update?`)를 호출할 때 항상 `false` 값을 반환하여 접근권한이 없게 되고, 또한, 모델과 연결되는 컨트롤러의 모든 액션들에 대해서 `authorize_action_for`와 같은 권한체크 메소드를 호출하게 될 때 역시 `false` 값을 반환하여 접근권한이 없게 된다.

언급했던 바와 같이 모델 클래스나 모델 클래스 객체에 대해서 권한체크를 할 때 뷰 파일내에서는 동사형과 형용사형의 메소드를 사용할 수 있다. 예를 들어 모델 클래스나 객체에 대해서 `update` 메소드에 대해서 권한을 물을 때 동사형인 `can_update?` 메소드나 형용사형인 `updatable_by?` 메소드 중에 아무거나 사용해도 된다. 단, 동사형의 메소드를 사용할 때는 인증용 모델인 `User` 클래스내에 `include Authority:UserAbilities`를 추가하고 형용사형의 메소드를 사용할 때는 대상 모델들의 클래스내에 `include Authority::Abilities`를 추가해야 한다.

특정 모델에 대해서 별도의 권한 로직은 구현하고자 할 때는, 모델이름과 `Authorizer`를 합성한 클래스 파일, 즉, 예를 들어 `Post` 모델에 대해서는 `PostAuthorizer` 클래스 파일에 정의하면 된다. 이 `Authorizer` 클래스 파일들은 `app/authorizers` 디렉토리에 위치한다.

만약 해당 모델에 대한 `Authorizer` 클래스 파일이 존재하지 않으면 최상위 `Authorizer` 클래스인 `ApplicationAuthorizer` 클래스에서 권한체크를 하게 된다.

현재 상태에서는 각 모델에 대한 `Authorizer` 클래스가 없기 때문에 모든 권한로직은 `ApplicationAuthorizer` 클래스에 집중되어 있다. 그러나 나중에 각각의 모델별로 권한로직을 분산할 수 있고 여러개의 모델이 동일한 `Authorizer` 클래스를 공유할 수 있도록 로직을 구성할 수 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/authority_strategy_zpsa6415ee4.png)

더 자세한 내용은 [`사용자 권한 설정하기`](https://luciuschoi.gitbooks.io/exploring_devise/content/apply_authentication/index.html)를 참고하기 바란다.

자, 이제 실제로 권한 로직을 구현해 보자.

여기서 구현할 권한 로직은 간단하다.

로그인하지 않은 상태에서 사용자는 `FoundblogApp`의 모든 게시글을 읽을 수만 있다(`:read`). 로그인한 상태에서 `:user` 권한을 가진 사용자는 댓글을 작성할 수 있고, 관리자로부터 `:author` 권한을 부여 받은 사용자는 새글을 작성할 수 있을 뿐만 아니라 본인이 작성한 글 또한  수정(`:update`)하거나 삭제(`:destroy`)할 수 있도록 하자. 물론 관리자 권한을 가진 사용자는 모든 게시물에 대해서 작성, 수정 및 삭제 권한을 가진다.

위의 권한 로직을 코드로 구현하면 아래와 같다.

최상위 권한 로직을 구현하기 위해서 `app/authorizers/application_authorizer.rb` 파일을 열고 아래와 같이 변경한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
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
{%endace%}


지금까지 작업한 내용을 로컬 저장소로 커밋한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git add .
$ git commit -m "제02장 8절 : authority 젬"
$ git tag "제02장8절"
{%endace%}


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장8절

---

_**References:**_

1. [nathanl/authority](https://github.com/nathanl/authority)
2. [Devise젬 파헤치기 - 사용자 권한 설정하기](http://luciuschoi.gitbooks.io/exploring_devise/user_authorization/README.html)
