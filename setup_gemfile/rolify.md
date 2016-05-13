# rolify 젬

[`rolify`](https://github.com/EppO/rolify) 젬을 사용하면 사용자별로 복수개의 모델에 대해서 권한을 쉽게 지정할 수 있도록 관련 모델과 다양한 메소드를 자동으로 만들어 준다.

우선 `Role`이라는 모델을 아래와 같이 생성한다. 이 때 `User` 모델이 이미 생성되어 있는 것으로 간주한다.

```bash
$ bin/rails g rolify Role                                                                       
Running via Spring preloader in process 94799
      invoke  active_record
      create    app/models/role.rb
      invoke    test_unit
      create      test/models/role_test.rb
      create      test/fixtures/roles.yml
      insert    app/models/role.rb
      create    db/migrate/20160513065619_rolify_create_roles.rb
      insert  app/models/user.rb
      create  config/initializers/rolify.rb
===============================================================================

An initializer file has been created here: config/initializers/rolify.rb, you
can change rolify settings to match your needs.
Defaults values are commented out.

A Role class has been created in app/models (with the name you gave as
argument otherwise the default is role.rb), you can add your own business logic
inside.

Inside your User class (or the name you gave as argument otherwise the default
is user.rb), rolify method has been inserted to provide rolify methods.
```

이 작업으로 인증 모델로 사용하는 `User` 모델 클래스 파일에 아래와 같이 자동으로 `rolify` 매크로 메소드가 추가된 것을 확인할 수 있다. (인증 대상이 되는 모델에는 `resourcify` 매크로 메소드를 직접 추가해 주어야 한다.)

> #### Caution::주의
> 
> 마이그레이션 작업을 하기 전에 한가지 주의해야 할 사항이 있다.
현재 설치된 `rolify`젬의 버전은 가장 최근 버전인 `5.1.0`이다. 그러나 버전 `3.4.0`을 사용할 경우에는 약간의 버그로 인하여 생성된 마이그레이션 파일의 파일명에서 `.rb`확장자가 누락되어 있어 마이그레이션 파일로 레일스가 인식하지 못한다. 따라서 파일명에 `.rb` 확장자을 붙이면 간단하게 문제를 해결할 수 있다.

이제 아래와 같이 마이그레이션 작업을 진행한다.

```bash
$ bin/rake db:migrate                                                                           
Running via Spring preloader in process 94895
== 20160513065619 RolifyCreateRoles: migrating ================================
-- create_table(:roles)
   -> 0.0287s
-- create_table(:users_roles, {:id=>false})
   -> 0.0095s
-- add_index(:roles, :name)
   -> 0.0147s
-- add_index(:roles, [:name, :resource_type, :resource_id])
   -> 0.0134s
-- add_index(:users_roles, [:user_id, :role_id])
   -> 0.0171s
== 20160513065619 RolifyCreateRoles: migrated (0.0838s) =======================
```

이 프로젝트에서 정의할 `Role`은 크게 세가지로 구분해 볼 수 있다.

|Role| 권한내용 |
|---|---|
|`:user` | 회원가입 후 이메일 인증절차가 완료된 상태에서 디폴트로 가지는 권한이다. 모든 글과 댓글에 대한 읽기 권한 및 댓글작성 권한도 동시에 가진다. 댓글은 수정할 수 없다. |
|`:author` | `:user` 권한에 더해서 글 작성 권한도 가진다. 본인이 작성한 글에 대해서는 수정 및 삭제 권한도 동시에 가진다. 관리자 권한을 가진 사용자가 특정 사용자에 대해서 `:author` 권한을 지정할 수 있다. |
|`:admin` | 관리자 권한으로 모든 권한을 가진다.|

회원가입시 인증 절차가 완료되면 해당 사용자에 대해서 `:user` 라는 `Role`을 지정해야 한다. 이를 위해서는 `confirm!` 인스턴스 메소드를 오버라이드하면 된다.

```ruby
class User < ActiveRecord::Base
  devise :database_authenticatable, :registerable,
     :recoverable, :rememberable, :trackable, :validatable, :confirmable

  def confirm!
    super
    add_role :user
  end
end
```

지금 구현한 내용을 확인해 보자.

어플리케이션을 시작하고(`rails s`), 레일스 콘솔을 아래와 같이 열어둔다.

```bash
$ bin/rails console -s
Running via Spring preloader in process 95548
Loading development environment in sandbox (Rails 4.2.6)
Any modifications you make will be rolled back on exit
irb(main):001:0>
```

> #### Note::노트
> 
> `-s` 옵션(sandbox)으로 콘솔을 열면 아래의 터미널 화면에서 1번으로 표시된 부분에서 현재 `sandbox` 모드임을 알 수 있고, 세션 중에 발생하는 모든 데이터 변경이 콘솔 종료시 모두 롤백되므로 간단한 테스트만 할 경우 매우 유용하다.


![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-12_10-06-12_zpsf529fe9a.png)

이제 콘솔에서 아래와 같이 사용자를 추가하면,

```bash
irb(main):001:0> User.create! email: "user1@email.com", password: "12345678"
```

> #### Caution::주의 
>
> `sand` 모드에서는 확인메일이 발송되지 않는다. 따라서 아래와 같이 확인메일을 테스트하기 위해서 `rails console`과 같이 일반 모드의 콘솔을 실행해야 한다. 

사용자가 추가되면서 확인 이메일이 브라우저상에 보여지게 된다. 그리고 아래의 화면과 같이 브라우저상의 이메일 내용 중에 보이는 `Confirm my account`(1번) 링크를 가입했던 사용자가 클릭하게 되면,

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-12_10-20-44_zpsd5c3b835.png)

아래와 같은 로그인 화면으로 이동하게 된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-12_10-43-18_zpsc0c2fff1.png)


이 때 위에서 `User` 모델 클래스에서 오버라이드했던 `confirm!` 메소드가 실행될 때 `add_role :user`가 실행된 것을 콘솔에서도 확인할 수 있다.

```bash
...

SQL (0.2ms)  INSERT INTO "roles" ("created_at", "name", "updated_at") VALUES (?, ?, ?)  [["created_at", "2014-06-12 01:39:21.166149"], ["name", "user"], ["updated_at", "2014-06-12 01:39:21.166149"]]
...

```

> #### Caution::주의
>
> 레일스 4.2.6에서는 `role` 추가시 `ArgumentError: Unknown key: :optional.` 에러가 발생할 수 있다. 이 때는 아래와 같이 `:optional => true` 코드라인을 삭제한다. 
> ```ruby
class Role < ActiveRecord::Base
  has_and_belongs_to_many :users, :join_table => :users_roles
>
  belongs_to :resource, 
             :polymorphic => true
>             
  validates :resource_type,
            :inclusion => { :in => Rolify.resource_types },
            :allow_nil => true
>
  scopify
end
```

이제 방금전에 가입했던 이메일로 로그인하면 `Home#index` 뷰 페이지가 브라우저상에 보여야 한다.

---

> **소스보기** https://github.com/LuciusChoi/foundblog/tree/제2.4장

---


_**References:**_

1. [EppO/rolify](https://github.com/EppO/rolify)
