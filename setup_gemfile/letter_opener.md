# letter_opener 젬

[`letter_opener`](https://github.com/ryanb/letter_opener) 젬은 개발환경에서 실제로 이메일을 보내는 대신에 브라우저에서 이메일을 볼 수 있도록 해 준다.

처음부터 작업을 따라 해온 경우라면 이미 `Gemfile` 파일에 아래와 같이 젬을 추가한 후 번들 인스톨한 상태다.

{%ace edit=false, lang='ruby', theme='monokai'%}
gem "letter_opener", :group => :development
{%endace%}

> ####Caution::주의
>
> `production` 환경에서는 이 젬을 사용하지 않을 것이기 때문에, `:group => :development` 옵션을 추가한다.

셋업을 하기 위해서는 `config/environments/development.rb` 파일을 열고 아래와 같이 추가한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
config.action_mailer.delivery_method = :letter_opener
{%endace%}

이제 회원 등록시 인증을 위한 이메일 브라우저에서 확인할 수 있을 것이다. 이를 위해서 레일스 로컬 서버를 실행한 후

{%ace edit=false, lang='sh', theme='monokai'%}
$ rails server
{%endace%}

브라우저에서 http://locahost:3000/users/sign_out 으로 접속하여 회원 등록을 하면 아래와 같은 화면을 보게 된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-10_11-22-34_zps5afcd5bc.png)

메일 내용 중 `Confirm my account` 링크를 클릭하면 `"Your email address has been successfully confirmed."` 메시지와 함께 로그인 창이 보이게 된다.

지금까지 작업한 내용을 로컬 저장소로 커밋한다.

{%ace edit=false, lang='sh', theme='monokai'%}
$ git add .
$ git commit -m "제02장 6절 : letter_opener 젬"
$ git tag "제02장6절"
{%endace%}

---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장6절

---

_**References:**_

1. [ryanb/letter_opener](https://github.com/ryanb/letter_opener)
