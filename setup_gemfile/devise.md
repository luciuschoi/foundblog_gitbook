# devise 젬

> #### Note::노트
>
> 사용자 인증과 권한 설정에 대해서는 [`Devise젬 파헤치기`](https://www.gitbook.io/book/luciuschoi/exploring_devise) 온라인 책을 참고하면 도움이 된다.

`devise` 젬은 사용자의 인증 서비스를 쉽게 구현할 수 있도록 도와준다.

우선 아래와 같이 `devise` 젬을 인스톨한다.

{%ace edit=true, lang='sh'%}
$ bin/rails g devise:install
Running via Spring preloader in process 93749
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. If you are deploying on Heroku with Rails 3.2 only, you may want to set:

       config.assets.initialize_on_precompile = false

     On config/application.rb forcing your application to not access the DB
     or load models when precompiling your assets.

  5. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

===============================================================================
{%endace%}

실행 결과 안내문에서 5가지의 추가조치 사항을 알려주는데, 이중에서 1, 2, 5번 내용을 반영하도록 한다.

**[1번 조치사항]** `config/environments/development.rb` 파일을 열고 아래와 같이 `default_url_options`을 지정한다.

{%ace edit=true, lang='ruby'%}
config.action_mailer.default_url_options = { host: 'localhost:3000' }
{%endace%}

아래에서 보게 될 것이지만, 사용자 가입 후 확인을 위한 이메일을 발송하게 될 것이기 때문에 실제로 이메일을 발송하도록 환경을 준비할 필요가 있다. 이에 대한 자세한 내용을 레일스 가이드의  [`Action Mailer Configuration`](http://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-configuration)을 참고하기 바란다. 그러나, 개발시에는 이메일을 실제로 보내는 것 보다는 브라우저에서 이메일 내용을 확인할 수 있으면 편리하다. 즉, `letter_opener` 젬을 이용하여 이와 같은 환경을 만들 수 있는데 이에 대해서는 [`여기`](/setup_gemfile/letter_opener.html)를 미리 참고하여 설정을 해 두도록 한다. 결국 아래와 같이 config 옵션을 지정하면 된다.

{%ace edit=true, lang='ruby'%}
config.action_mailer.default_url_options = { host: 'localhost:3000' }
config.action_mailer.delivery_method = :letter_opener
{%endace%}

> #### Caution::주의
>
> `production` 환경(`production.rb`)에서는 어플리케이션의 실제 호스트의 이름으로 `:host`를 변경해 주어야 한다.


**[2번 조치사항]** `home` 컨트롤러 및 `index` 액션을 추가해 주어야 하며 아래와 같이 실행한 후,

{%ace edit=true, lang='sh'%}
$ bin/rails g controller home index                                                                             master
Running via Spring preloader in process 93974
      create  app/controllers/home_controller.rb
       route  get 'home/index'
      invoke  erb
      create    app/views/home
      create    app/views/home/index.html.erb
      invoke  test_unit
      create    test/controllers/home_controller_test.rb
      invoke  helper
      create    app/helpers/home_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/home.coffee
      invoke    scss
      create      app/assets/stylesheets/home.scss
{%endace%}

`config/routes.rb` 파일에 `root`를 지정한다.

{%ace edit=true, lang='ruby'%}
Rails.application.routes.draw do
  root 'home#index'
  # get 'home/index'
end
{%endace%}

사실 `home` 컨트롤러는 나중에 필요없게 된다. 블로그의 루트는 결국 `posts#index` 액션으로 지정할 것이기 때문이다. 따라서 이부분은 생략할 수 있다.


**[3번 조치사항]** `flash` 메시지를 어플리케이션 레이아웃 파일에 추가하기 위해서 `app/views/layouts/application.html.erb` 파일을 열고 `<body></body>` 태그 사이에 아래와 같이 임시로 작성해 둔다. 이것은 나중에 다시 구체적으로 작업을 할 예정이다.

{%ace edit=true, lang='rhtml'%}
<body>

  <p class="notice"><%= notice %></p>
  <p class="alert"><%= alert %></p>

  <%= yield %>

</body>
{%endace%}

**[4번 조치사항]** 사실 4번째 조치사항은 레일스 3.2 프로젝트를 허로쿠에 배포할 경우에만 국한 된 사항이므로 여기서는 적용을 하지 않을 것이다. 그러나, 필요한 상황에서는 `config/application.rb` 파일을 열고 아래와 같이 옵션을 추가한다.

{%ace edit=true, lang='ruby'%}
config.assets.initialize_on_precompile = false
{%endace%}

**[5번 조치사항]**  5번 조치사항은 `devise`에서 제공해는 다양한 폼을 개발자의 의도에 맞게 수정하기 위해서 `app/views/` 디렉토리에 `devise`라는 하위디렉토리를 생성하고 관련 폼 뷰 템플릿 파일들을 생성한다.

{%ace edit=true, lang='sh'%}
$ bin/rails g devise:views                                                                     
Running via Spring preloader in process 94426
      invoke  Devise::Generators::SharedViewsGenerator
      create    app/views/devise/shared
      create    app/views/devise/shared/_links.html.erb
      invoke  simple_form_for
      create    app/views/devise/confirmations
      create    app/views/devise/confirmations/new.html.erb
      create    app/views/devise/passwords
      create    app/views/devise/passwords/edit.html.erb
      create    app/views/devise/passwords/new.html.erb
      create    app/views/devise/registrations
      create    app/views/devise/registrations/edit.html.erb
      create    app/views/devise/registrations/new.html.erb
      create    app/views/devise/sessions
      create    app/views/devise/sessions/new.html.erb
      create    app/views/devise/unlocks
      create    app/views/devise/unlocks/new.html.erb
      invoke  erb
      create    app/views/devise/mailer
      create    app/views/devise/mailer/confirmation_instructions.html.erb
      create    app/views/devise/mailer/password_change.html.erb
      create    app/views/devise/mailer/reset_password_instructions.html.erb
      create    app/views/devise/mailer/unlock_instructions.html.erb
{%endace%}

이상으로 `devise` 젬을 설치한 후 필요한 조치에 대해서 설명을 했다.

다음으로는 `devise` 제너레이터를 이용하여 `User` 모델 리소스를 생성한다.

{%ace edit=true, lang='sh'%}
$ bin/rails g devise User  
Running via Spring preloader in process 94459
      invoke  active_record
      create    db/migrate/20160513064912_devise_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
      insert    app/models/user.rb
       route  devise_for :users
{%endace%}

마이그레이션 전에 한가지 추가 작업을 해야 한다. 이 프로젝트에서는 사용자 등록시에 확인용 이메일을 발송하여 해당 사용자가 받은 이메일의 내용 중에 본인 인증을 위한 링크를 클릭하므로써 절차를 마무리하는 로직을 구현하고자한다.

이를 위해서 `app/models/user.rb` 파일을 열고 아래와 같이 코멘트 처리되어 있는 `:confirmable`을 `devise` 메소드의 인수로 추가해 준다.

{%ace edit=true, lang='ruby'%}
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :confirmable
end
{%endace%}

그리고 `db/migrate/20140609040948_devise_create_users.rb` 파일을 열고 `##Confirmable` 아래 4개의 코드라인과 하단의 `:confirmable_token` 속성에 대한 `add_index` 메소드 코드라인의 코멘트 문자를 제거한다.

{%ace edit=true, lang='ruby'%}

## Confirmable
 t.string   :confirmation_token
 t.datetime :confirmed_at
 t.datetime :confirmation_sent_at
 t.string   :unconfirmed_email # Only if using reconfirmable
...
 add_index :users, :confirmation_token,   unique: true
---
{%endace%}

이제 마이그레이션 작업을 한다.

먼저 데이터베이스를 먼저 생성해 주어야 한다.
{%ace edit=true, lang='sh'%}
$ bin/rake db:create                                                                            
Running via Spring preloader in process 94740
{%endace%}

그리고 `db:migrate` 작업을 실행한다.

{%ace edit=true, lang='sh'%}
$ bin/rake db:migrate                                                                           
Running via Spring preloader in process 94767
== 20160513064912 DeviseCreateUsers: migrating ================================
-- create_table(:users)
   -> 0.0275s
-- add_index(:users, :email, {:unique=>true})
   -> 0.0112s
-- add_index(:users, :reset_password_token, {:unique=>true})
   -> 0.0123s
-- add_index(:users, :confirmation_token, {:unique=>true})
   -> 0.0112s
== 20160513064912 DeviseCreateUsers: migrated (0.0624s) =======================
{%endace%}

이제 어플리케이션내의 모든 컨트롤러와 뷰 파일에서 아래의 헬퍼 메소드를 사용할 수 있게 된다.

* `authenticate_user!`
* `current_user`
* `user_signed_in?`
* `user_session`

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02.3장

---

_**References:**_

1. [plataformatec/devise](https://github.com/plataformatec/devise)
