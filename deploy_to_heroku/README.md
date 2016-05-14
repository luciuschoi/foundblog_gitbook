# 허로쿠에 배포하기

지금까지 작업한 내용을 허로쿠에 배포해 보자.

배포 전에 한가지 해야 할 것이 있다. `devise` 젬의 `confirmable` 기능을 사용하여 사용자 등록시 확인 이메일을 발송해야 한다. 이를 위해서 이메일 발송을 위한 세팅을 해야 한다.

`config/environments/production.rb` 파일을 열고 아래와 같이 추가해 준다.

{%ace edit=true, lang='ruby'%}
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address: ENV['SMTP_ADDRESS'],
  port: ENV['SMTP_PORT'],
  authentication: "plain",
  user_name: ENV['SMTP_USERNAME'],
  password: ENV['SMTP_PASSWORD'],
  enable_starttls_auto: false
}
config.action_mailer.raise_delivery_errors = true
{%endace%}

이를 위해서 `Gemfile`에 두개의 젬을 추가하고

{%ace edit=true, lang='ruby'%}
gem 'pg', group: :production
gem 'rails_12factor', group: :production
{%endace%}

번들 인스톨한다.

{%ace edit=true, lang='sh'%}
$ bin/bundle install
{%endace%}

아직 `github`로 `git push` 하지 않았다면 `github`에서 저장소를 만들고 아래와 같이 작업한다.

{%ace edit=true, lang='sh'%}
$ git remote add origin https://github.com/[github-account]/foundblog.git
$ git push -u origin master
{%endace%}

이제 [허로쿠 툴벨트](https://toolbelt.heroku.com)를 설치하고, 허로쿠 `login`한 후 `app`를 생성한다.

{%ace edit=true, lang='sh'%}
$ heroku login
$ heroku apps:create example
Creating example... done, stack is cedar
http://example.herokuapp.com/ | git@heroku.com:example.git
{%endace%}

이미 "foundblog`라는 `app` 이름은 데모 어플리케이션으로 사용했기 때문에 다른 이름을 사용해야 한다.

다음은 heroku-postgresql을 아래와 같이 추가하고,

{%ace edit=true, lang='sh'%}
$ heroku addons:add heroku-postgresql
{%endace%}

다음과 같이 허로쿠로 배포한 후,

{%ace edit=true, lang='sh'%}
$ git push heroku master
{%endace%}

아래와 같이 데이터베이스 마이그레이션을 해 주면 배포가 완성된다.

{%ace edit=true, lang='sh'%}
$ heroku run rake db:migrate
{%endace%}

다음 이메일 발송을 위한 환경변수를 허로쿠로 등록해야 한다.

{%ace edit=true, lang='sh'%}
$ heroku config:set SMTP_ADDRESS=[mail_domain_name]
$ heroku config:set SMTP_PORT=[port_number]
$ heroku config:set SMTP_USERNAME=[username]
$ heroku config:set SMTP_PASSWORD=[password]
{%endace%}

`[]` 안에 각자의 계정에 맞는 값을 입력한다.

허로쿠의 로그 상태를 모니터링하고자 할 때는 아래와 같은 명령을 실행한다.

{%ace edit=true, lang='sh'%}
$ heroku logs --tail
{%endace%}

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제12장
