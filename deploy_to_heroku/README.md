# 허로쿠에 배포하기

지금까지 작업한 내용을 [허로쿠](https://www.heroku.com/)로 배포할 것이다.

배포 전에 해야 할 것들이 있다. `devise` 젬의 `confirmable` 기능을 사용할 경우, 사용자 등록시 확인 이메일을 발송한다. 이를 위해서 이메일 발송을 위한 셋팅을 해야 한다.

여기서는 [`Mailgun`](https://mailgun.com/)을 사용해서 메일을 발송할 것이다.

우선, `mailgun_rails` 젬을 추가하고 번들 인스톨한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
gem 'mailgun_rails'
{%endace%}

`config/environments/production.rb` 파일을 열고 아래와 같이 추가한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
config.action_mailer.delivery_method = :mailgun
config.action_mailer.mailgun_settings = {
  api_key: Rails.application.secrets.mailgun_api_key,
  domain: 'rorlab.org'
}
{%endace%}

> ####Note::노트
>
> `domain` 값은 `Mailgun` 설정시에 사용한 도메인명을 지정한다.

`config/secrests.yml` 파일을 열고 아래와 같이  `mailgun_api_key` 키를 등록한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
  mailgun_api_key: <%= ENV["MAILGUN_API_KEY"] %>
{%endace%}

실제 키값을 `MAILGUN_API_KEY` 환경변수로 허로쿠에 등록한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ heroku config:set MAILGUN_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{%endace%}

허로쿠 배포를 위해서는 `Gemfile`에 아래의 두 젬을 필수로 추가하고 번들 인스톨한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
gem 'pg', group: :production
gem 'rails_12factor', group: :production
{%endace%}

아직 `github`로 `git push` 하지 않았다면 `github`에서 저장소를 만들고 푸시한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git remote add origin https://github.com/[github-account]/foundblog_app.git
$ git push -u origin master
{%endace%}

이제 [허로쿠 툴벨트](https://toolbelt.heroku.com)를 설치하고, 허로쿠 `login`한 후 애플리케이션을 생성한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ heroku login
$ heroku create
Heroku CLI submits usage information back to Heroku. If you would like to disable this, set `skip_analytics: true` in /Users/[your-account]/.heroku/config.json
Creating app... done, ⬢ obscure-cove-70618
https://obscure-cove-70618.herokuapp.com/ | https://git.heroku.com/obscure-cove-70618.git
{%endace%}

> ####Note::노트
>
> `heroku create` 명령 실행시 애플리케이션 이름을 지정할 수 있다. 그러나 중복된 이름을 지정하면 아래와 같이 에러 메시지 표시된다. 위와 같이 애플리케이션 이름을 지정하지 않으면 허로쿠가 알아서 임의로 지정한다.
>
{%ace edit=false, lang='sh', theme='monokai'%}
$ heroku create foundblog                                                                                        
Creating ⬢ foundblog... !!!
 ▸    Name is already taken
{%endace%}


허로쿠 애플리케이션 이름을 `foundblog6`로 변경한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ heroku apps:rename foundblog6                        
Renaming obscure-cove-70618 to foundblog6... done
https://foundblog6.herokuapp.com/ | https://git.heroku.com/foundblog6.git
Git remote heroku updated
 ▸    Don't forget to update git remotes for all other local checkouts of the app.
{%endace%}

그리고, 변경된 도메인 주소를 `config/routes.rb` 파일에서 `default_url_options[:host]`로 지정한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
Rails.application.routes.default_url_options[:host] = 'foundblog6.herokuapp.com'
{%endace%}

다음은 `heroku-postgresql`을 아래와 같이 추가하고,

{%ace edit=false, lang='sh', theme='monokai'%}
$ heroku addons:create heroku-postgresql
Creating postgresql-rigid-15787... done, (free)
Adding postgresql-rigid-15787 to obscure-cove-70618... done
Setting DATABASE_URL and restarting obscure-cove-70618... done, v3
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Use `heroku addons:docs heroku-postgresql` to view documentation.
{%endace%}

다음과 같이 허로쿠로 배포한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git push heroku master
Counting objects: 384, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (354/354), done.
Writing objects: 100% (384/384), 461.12 KiB | 0 bytes/s, done.
Total 384 (delta 144), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Ruby app detected
remote: -----> Compiling Ruby/Rails
remote: -----> Using Ruby version: ruby-2.2.4
remote: -----> Installing dependencies using bundler 1.11.2
remote:        Running: bundle install --without development:test --path vendor/bundle --binstubs vendor/bundle/bin -j4 --deployment
remote:        Fetching gem metadata from https://rubygems.org/...........
remote:        Fetching version metadata from https://rubygems.org/...
remote:        Fetching dependency metadata from https://rubygems.org/..
remote:        Installing i18n 0.7.0
remote:        Installing rake 11.1.2
remote:        Installing json 1.8.3 with native extensions
remote:        Installing minitest 5.8.4
remote:        Installing thread_safe 0.3.5
remote:        Installing builder 3.2.2
remote:        Installing mini_portile2 2.0.0
remote:        Installing erubis 2.7.0
remote:        Installing mime-types-data 3.2016.0221
remote:        Installing rack 1.6.4
remote:        Installing arel 6.0.3
remote:        Installing execjs 2.6.0
remote:        Installing babel-source 5.8.35
remote:        Installing bcrypt 3.1.11 with native extensions
remote:        Installing coffee-script-source 1.10.0
remote:        Installing thor 0.19.1
remote:        Installing orm_adapter 0.5.0
remote:        Installing concurrent-ruby 1.0.2
remote:        Installing tilt 2.0.3
remote:        Installing sass 3.4.22
remote:        Installing multi_json 1.12.0
remote:        Installing mysql2 0.4.4 with native extensions
remote:        Installing polyglot 0.3.5
remote:        Using bundler 1.11.2
remote:        Installing pg 0.18.4 with native extensions
remote:        Installing rails_serve_static_assets 0.0.5
remote:        Installing rails_stdout_logging 0.0.5
remote:        Installing rolify 5.1.0
remote:        Installing tzinfo 1.2.2
remote:        Installing nokogiri 1.6.7.2 with native extensions
remote:        Installing mime-types 3.0
remote:        Installing rack-test 0.6.3
remote:        Installing warden 1.2.6
remote:        Installing rdoc 4.2.2
remote:        Installing uglifier 3.0.0
remote:        Installing babel-transpiler 0.7.0
remote:        Installing coffee-script 2.4.1
remote:        Installing sprockets 3.6.0
remote:        Installing treetop 1.6.5
remote:        Installing rails_12factor 0.0.3
remote:        Installing activesupport 4.2.6
remote:        Installing mail 2.6.4
remote:        Installing sdoc 0.4.1
remote:        Installing sprockets-es6 0.9.0
remote:        Installing search_cop 1.0.6
remote:        Installing rails-deprecated_sanitizer 1.0.3
remote:        Installing globalid 0.3.6
remote:        Installing activemodel 4.2.6
remote:        Installing authority 3.1.0
remote:        Installing jbuilder 2.4.1
remote:        Installing activejob 4.2.6
remote:        Installing activerecord 4.2.6
remote:        Installing acts-as-taggable-on 3.5.0
remote:        Installing loofah 2.0.3
remote:        Installing rails-dom-testing 1.0.7
remote:        Installing rails-html-sanitizer 1.0.3
remote:        Installing actionview 4.2.6
remote:        Installing actionpack 4.2.6
remote:        Installing actionmailer 4.2.6
remote:        Installing sprockets-rails 3.0.4
remote:        Installing railties 4.2.6
remote:        Installing simple_form 3.2.1
remote:        Installing coffee-rails 4.1.1
remote:        Installing responders 2.2.0
remote:        Installing sass-rails 5.0.4
remote:        Installing foundation-rails 6.2.1.0
remote:        Installing jquery-rails 4.1.1
remote:        Installing turbolinks 2.5.3
remote:        Installing rails 4.2.6
remote:        Installing devise 4.1.0
remote:        Installing foundation-icons-sass-rails 3.0.0
remote:        Installing jquery-turbolinks 2.1.0
remote:        Bundle complete! 24 Gemfile dependencies, 72 gems now installed.
remote:        Gems in the groups development and test were not installed.
remote:        Bundled gems are installed into ./vendor/bundle.
remote:        Post-install message from rdoc:
remote:        Depending on your version of ruby, you may need to install ruby rdoc/ri data:
remote:        <= 1.8.6 : unsupported
remote:        = 1.8.7 : gem install rdoc-data; rdoc-data --install
remote:        = 1.9.1 : gem install rdoc-data; rdoc-data --install
remote:        >= 1.9.2 : nothing to do! Yay!
remote:        Post-install message from acts-as-taggable-on:
remote:        When upgrading
remote:        Re-run the migrations generator
remote:        rake acts_as_taggable_on_engine:install:migrations
remote:        This will create any new migrations and skip existing ones
remote:        Version 3.5.0 has a migration for mysql adapter
remote:        Bundle completed (33.37s)
remote:        Cleaning up the bundler cache.
remote: -----> Preparing app for Rails asset pipeline
remote:        Running: rake assets:precompile
remote:        I, [2016-05-17T11:36:36.420804 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/blog_header-a5461c7eb9f3a10fb735d9b1b0e8722170fedc169431b57b83089a3c8a71d0a5.png
remote:        I, [2016-05-17T11:36:36.423966 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundation_blog_emblem-760826bb128e69721abc93eed446e0ebc445579095d008aa88ebf44b3ef4adf8.png
remote:        I, [2016-05-17T11:36:36.426576 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundblog_logo-f8191670608ca09207632f88a4c4cb7527b1e5f72addb1f7b3dddba60a4ec22a.png
remote:        I, [2016-05-17T11:37:00.098804 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/application-a14ff9e8b240c5f5ce4c70395309397303bfd4b439f34ded53005639bf380756.js
remote:        I, [2016-05-17T11:37:00.098983 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/application-a14ff9e8b240c5f5ce4c70395309397303bfd4b439f34ded53005639bf380756.js.gz
remote:        DEPRECATION WARNING on line 81 of /tmp/build_e5b54080344c5bdfb909063b1cec9956/vendor/bundle/ruby/2.2.0/gems/foundation-rails-6.2.1.0/vendor/assets/scss/components/_button-group.scss: #{} interpolation near operators will be simplified
remote:        in a future version of Sass. To preserve the current behavior, use quotes:
remote:        unquote('#{$buttongroup-spacing} == "0"')
remote:        You can use the sass-convert command to automatically fix most cases.
remote:        I, [2016-05-17T11:37:04.447656 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/application-9281e677837952fcd011b81e89a56ec47aa02ebf6f799b90dbfda282cd5ea59b.css
remote:        I, [2016-05-17T11:37:04.448230 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/application-9281e677837952fcd011b81e89a56ec47aa02ebf6f799b90dbfda282cd5ea59b.css.gz
remote:        I, [2016-05-17T11:37:04.448678 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundation-icons-9189cd8788a2d42f89ecb72f08d55cc366a3abc441c3413d9ceca66ec3144e46.eot
remote:        I, [2016-05-17T11:37:04.449693 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundation-icons-9189cd8788a2d42f89ecb72f08d55cc366a3abc441c3413d9ceca66ec3144e46.eot.gz
remote:        I, [2016-05-17T11:37:04.450240 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundation-icons-8c44c3feedae5331a281278ea3ba91d2255928a2f3010d316d6fbb9052e0c2ec.woff
remote:        I, [2016-05-17T11:37:04.455020 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundation-icons-7e1dd03dd4ce90b658052554cd7459df16716717389a552fa4c6d56a5f8933e6.ttf
remote:        I, [2016-05-17T11:37:04.455504 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundation-icons-7e1dd03dd4ce90b658052554cd7459df16716717389a552fa4c6d56a5f8933e6.ttf.gz
remote:        I, [2016-05-17T11:37:04.455921 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundation-icons-ba2d122321cfede034deb120f35abc790a023d29065c36bd53e57039897ad052.svg
remote:        I, [2016-05-17T11:37:04.459644 #1754]  INFO -- : Writing /tmp/build_e5b54080344c5bdfb909063b1cec9956/public/assets/foundation-icons-ba2d122321cfede034deb120f35abc790a023d29065c36bd53e57039897ad052.svg.gz
remote:        Asset precompilation completed (30.13s)
remote:        Cleaning assets
remote:        Running: rake assets:clean
remote:
remote: ###### WARNING:
remote:        You have not declared a Ruby version in your Gemfile.
remote:        To set your Ruby version add this line to your Gemfile:
remote:        ruby '2.2.4'
remote:        # See https://devcenter.heroku.com/articles/ruby-versions for more information.
remote:
remote: ###### WARNING:
remote:        No Procfile detected, using the default web server.
remote:        We recommend explicitly declaring how to boot your server process via a Procfile.
remote:        https://devcenter.heroku.com/articles/ruby-default-web-server
remote:
remote: -----> Discovering process types
remote:        Procfile declares types     -> (none)
remote:        Default types for buildpack -> console, rake, web, worker
remote:
remote: -----> Compressing...
remote:        Done: 34.9M
remote: -----> Launching...
remote:        Released v5
remote:        https://obscure-cove-70618.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy.... done.
To https://git.heroku.com/obscure-cove-70618.git
 * [new branch]      master -> master
{%endace%}

위의 배포 로그 중에서 2개의 경고문을 주의해서 봐야 한다.
우선, `Gemfile` 상단에 루비 버전을 명시해 주어야 한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
source 'https://rubygems.org'
ruby '2.3.0'
# Added for this project
gem 'foundation-rails'
gem 'simple_form'
gem 'devise'
gem 'rolify'
...
{%endace%}

두번째는 프로젝트 디렉토리에 `Procfile`을 작성해야 한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ vi Procfile
web: bundle exec rackup config.ru -p $PORT
{%endace%}


이제 방금 변경한 내용을 커밋한 후 `git push heroku master` 명령을 실행하면 아래와 같이 경고문 없이 배포되어야 한다.

{%ace edit=false, lang='sh', theme='monokai'%}
...
...
remote:        Bundle complete! 24 Gemfile dependencies, 72 gems now installed.
remote:        Gems in the groups development and test were not installed.
remote:        Bundled gems are installed into ./vendor/bundle.
remote:        Bundle completed (0.43s)
remote:        Cleaning up the bundler cache.
remote: -----> Preparing app for Rails asset pipeline
remote:        Running: rake assets:precompile
remote:        Asset precompilation completed (2.23s)
remote:        Cleaning assets
remote:        Running: rake assets:clean
remote:
remote:
remote: -----> Discovering process types
remote:        Procfile declares types     -> web
remote:        Default types for buildpack -> console, rake, worker
remote:
remote: -----> Compressing...
remote:        Done: 34.9M
remote: -----> Launching...
remote:        Released v8
remote:        https://obscure-cove-70618.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/obscure-cove-70618.git
   4b60169..7871b93  master -> master   
{%endace%}


이제, 아래와 같이 데이터베이스 마이그레이션을 해 주면 배포가 완성된다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ heroku run rake db:migrate                                                                                     master
Running rake db:migrate on obscure-cove-70618... up, run.3533
   (8.9ms)  CREATE TABLE "schema_migrations" ("version" character varying NOT NULL)
   (3.8ms)  CREATE UNIQUE INDEX  "unique_schema_migrations" ON "schema_migrations"  ("version")
  ActiveRecord::SchemaMigration Load (1.4ms)  SELECT "schema_migrations".* FROM "schema_migrations"
Migrating to DeviseCreateUsers (20160513064912)
   (1.3ms)  BEGIN
== 20160513064912 DeviseCreateUsers: migrating ================================
-- create_table(:users)
   (11.0ms)  CREATE TABLE "users" ("id" serial primary key, "email" character varying DEFAULT '' NOT NULL, "encrypted_password" character varying DEFAULT '' NOT NULL, "reset_password_token" character varying, "reset_password_sent_at" timestamp, "remember_created_at" timestamp, "sign_in_count" integer DEFAULT 0 NOT NULL, "current_sign_in_at" timestamp, "last_sign_in_at" timestamp, "current_sign_in_ip" character varying, "last_sign_in_ip" character varying, "confirmation_token" character varying, "confirmed_at" timestamp, "confirmation_sent_at" timestamp, "unconfirmed_email" character varying, "created_at" timestamp NOT NULL, "updated_at" timestamp NOT NULL)
   -> 0.0188s
-- add_index(:users, :email, {:unique=>true})
   (4.9ms)  CREATE UNIQUE INDEX  "index_users_on_email" ON "users"  ("email")
   -> 0.0088s
-- add_index(:users, :reset_password_token, {:unique=>true})
   (7.9ms)  CREATE UNIQUE INDEX  "index_users_on_reset_password_token" ON "users"  ("reset_password_token")
   -> 0.0114s
-- add_index(:users, :confirmation_token, {:unique=>true})
   (3.1ms)  CREATE UNIQUE INDEX  "index_users_on_confirmation_token" ON "users"  ("confirmation_token")
   -> 0.0065s
== 20160513064912 DeviseCreateUsers: migrated (0.0460s) =======================

  SQL (1.3ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160513064912"]]
   (3.2ms)  COMMIT
Migrating to RolifyCreateRoles (20160513065619)
   (1.1ms)  BEGIN
== 20160513065619 RolifyCreateRoles: migrating ================================
-- create_table(:roles)
   (7.6ms)  CREATE TABLE "roles" ("id" serial primary key, "name" character varying, "resource_id" integer, "resource_type" character varying, "created_at" timestamp, "updated_at" timestamp)
   -> 0.0083s
-- create_table(:users_roles, {:id=>false})
   (1.6ms)  CREATE TABLE "users_roles" ("user_id" integer, "role_id" integer)
   -> 0.0018s
-- add_index(:roles, :name)
   (3.0ms)  CREATE  INDEX  "index_roles_on_name" ON "roles"  ("name")
   -> 0.0065s
-- add_index(:roles, [:name, :resource_type, :resource_id])
   (3.3ms)  CREATE  INDEX  "index_roles_on_name_and_resource_type_and_resource_id" ON "roles"  ("name", "resource_type", "resource_id")
   -> 0.0067s
-- add_index(:users_roles, [:user_id, :role_id])
   (3.2ms)  CREATE  INDEX  "index_users_roles_on_user_id_and_role_id" ON "users_roles"  ("user_id", "role_id")
   -> 0.0066s
== 20160513065619 RolifyCreateRoles: migrated (0.0303s) =======================

  SQL (1.1ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160513065619"]]
   (2.4ms)  COMMIT
Migrating to CreateCategories (20160513091011)
   (1.1ms)  BEGIN
== 20160513091011 CreateCategories: migrating =================================
-- create_table(:categories)
   (6.2ms)  CREATE TABLE "categories" ("id" serial primary key, "user_id" integer, "name" character varying NOT NULL, "created_at" timestamp NOT NULL, "updated_at" timestamp NOT NULL)
   (3.0ms)  CREATE  INDEX  "index_categories_on_user_id" ON "categories"  ("user_id")
   (2.8ms)  ALTER TABLE "categories" ADD CONSTRAINT "fk_rails_b8e2f7adfc"
FOREIGN KEY ("user_id")
  REFERENCES "users" ("id")

   -> 0.0166s
== 20160513091011 CreateCategories: migrated (0.0167s) ========================

  SQL (1.1ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160513091011"]]
   (2.6ms)  COMMIT
Migrating to CreatePosts (20160513091050)
   (1.1ms)  BEGIN
== 20160513091050 CreatePosts: migrating ======================================
-- create_table(:posts)
   (9.1ms)  CREATE TABLE "posts" ("id" serial primary key, "category_id" integer, "user_id" integer, "title" character varying NOT NULL, "content" text NOT NULL, "published" boolean DEFAULT 'f', "created_at" timestamp NOT NULL, "updated_at" timestamp NOT NULL)
   (8.3ms)  CREATE  INDEX  "index_posts_on_category_id" ON "posts"  ("category_id")
   (3.2ms)  CREATE  INDEX  "index_posts_on_user_id" ON "posts"  ("user_id")
   (2.1ms)  ALTER TABLE "posts" ADD CONSTRAINT "fk_rails_9b1b26f040"
FOREIGN KEY ("category_id")
  REFERENCES "categories" ("id")

   (2.0ms)  ALTER TABLE "posts" ADD CONSTRAINT "fk_rails_5b5ddfd518"
FOREIGN KEY ("user_id")
  REFERENCES "users" ("id")

   -> 0.0351s
== 20160513091050 CreatePosts: migrated (0.0353s) =============================

  SQL (1.2ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160513091050"]]
   (3.0ms)  COMMIT
Migrating to CreateComments (20160513091124)
   (1.2ms)  BEGIN
== 20160513091124 CreateComments: migrating ===================================
-- create_table(:comments)
   (6.6ms)  CREATE TABLE "comments" ("id" serial primary key, "user_id" integer, "post_id" integer, "body" text, "created_at" timestamp NOT NULL, "updated_at" timestamp NOT NULL)
   (3.2ms)  CREATE  INDEX  "index_comments_on_user_id" ON "comments"  ("user_id")
   (3.8ms)  CREATE  INDEX  "index_comments_on_post_id" ON "comments"  ("post_id")
   (2.2ms)  ALTER TABLE "comments" ADD CONSTRAINT "fk_rails_03de2dc08c"
FOREIGN KEY ("user_id")
  REFERENCES "users" ("id")

   (2.3ms)  ALTER TABLE "comments" ADD CONSTRAINT "fk_rails_2fd19c0db7"
FOREIGN KEY ("post_id")
  REFERENCES "posts" ("id")

   -> 0.0276s
== 20160513091124 CreateComments: migrated (0.0300s) ==========================

  SQL (1.2ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160513091124"]]
   (2.8ms)  COMMIT
Migrating to ActsAsTaggableOnMigration (20160517073753)
   (1.2ms)  BEGIN
== 20160517073753 ActsAsTaggableOnMigration: migrating ========================
-- create_table(:tags)
   (7.2ms)  CREATE TABLE "tags" ("id" serial primary key, "name" character varying)
   -> 0.0080s
-- create_table(:taggings)
   (9.7ms)  CREATE TABLE "taggings" ("id" serial primary key, "tag_id" integer, "taggable_id" integer, "taggable_type" character varying, "tagger_id" integer, "tagger_type" character varying, "context" character varying(128), "created_at" timestamp)
   -> 0.0102s
-- add_index(:taggings, :tag_id)
   (3.2ms)  CREATE  INDEX  "index_taggings_on_tag_id" ON "taggings"  ("tag_id")
   -> 0.0070s
-- add_index(:taggings, [:taggable_id, :taggable_type, :context])
   (3.4ms)  CREATE  INDEX  "index_taggings_on_taggable_id_and_taggable_type_and_context" ON "taggings"  ("taggable_id", "taggable_type", "context")
   -> 0.0073s
== 20160517073753 ActsAsTaggableOnMigration: migrated (0.0330s) ===============

  SQL (1.2ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160517073753"]]
   (3.0ms)  COMMIT
Migrating to AddMissingUniqueIndices (20160517073754)
   (1.2ms)  BEGIN
== 20160517073754 AddMissingUniqueIndices: migrating ==========================
-- add_index(:tags, :name, {:unique=>true})
   (4.0ms)  CREATE UNIQUE INDEX  "index_tags_on_name" ON "tags"  ("name")
   -> 0.0100s
-- remove_index(:taggings, :tag_id)
   (2.0ms)  DROP INDEX "index_taggings_on_tag_id"
   -> 0.0045s
-- remove_index(:taggings, [:taggable_id, :taggable_type, :context])
   (2.2ms)  DROP INDEX "index_taggings_on_taggable_id_and_taggable_type_and_context"
   -> 0.0042s
-- add_index(:taggings, [:tag_id, :taggable_id, :taggable_type, :context, :tagger_id, :tagger_type], {:unique=>true, :name=>"taggings_idx"})
   (3.9ms)  CREATE UNIQUE INDEX  "taggings_idx" ON "taggings"  ("tag_id", "taggable_id", "taggable_type", "context", "tagger_id", "tagger_type")
   -> 0.0074s
== 20160517073754 AddMissingUniqueIndices: migrated (0.0265s) =================

  SQL (1.3ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160517073754"]]
   (5.2ms)  COMMIT
Migrating to AddTaggingsCounterCacheToTags (20160517073755)
   (1.1ms)  BEGIN
== 20160517073755 AddTaggingsCounterCacheToTags: migrating ====================
-- add_column(:tags, :taggings_count, :integer, {:default=>0})
   (8.7ms)  ALTER TABLE "tags" ADD "taggings_count" integer DEFAULT 0
   -> 0.0112s
  ActsAsTaggableOn::Tag Load (3.1ms)  SELECT  "tags".* FROM "tags"  ORDER BY "tags"."id" ASC LIMIT 1000
== 20160517073755 AddTaggingsCounterCacheToTags: migrated (0.0253s) ===========

  SQL (1.2ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160517073755"]]
   (8.1ms)  COMMIT
Migrating to AddMissingTaggableIndex (20160517073756)
   (1.2ms)  BEGIN
== 20160517073756 AddMissingTaggableIndex: migrating ==========================
-- add_index(:taggings, [:taggable_id, :taggable_type, :context])
   (3.6ms)  CREATE  INDEX  "index_taggings_on_taggable_id_and_taggable_type_and_context" ON "taggings"  ("taggable_id", "taggable_type", "context")
   -> 0.0074s
== 20160517073756 AddMissingTaggableIndex: migrated (0.0076s) =================

  SQL (1.1ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160517073756"]]
   (2.1ms)  COMMIT
Migrating to ChangeCollationForTagNames (20160517073757)
   (1.1ms)  BEGIN
== 20160517073757 ChangeCollationForTagNames: migrating =======================
== 20160517073757 ChangeCollationForTagNames: migrated (0.0004s) ==============

  SQL (1.1ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1)  [["version", "20160517073757"]]
   (1.9ms)  COMMIT
{%endace%}

허로쿠의 로그 상태를 모니터링하고자 할 때는 아래와 같은 명령을 실행한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ heroku logs --tail
{%endace%}


지금까지 작업한 내용을 로컬 저장소로 커밋한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git add .
$ git commit -m "제12장 : 허로쿠에 배포하기"
$ git tag "제12장"
{%endace%}

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제12장
