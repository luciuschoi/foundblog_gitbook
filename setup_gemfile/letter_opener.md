# letter_opener 젬

[`letter_opener`](https://github.com/ryanb/letter_opener) 젬은 개발환경에서 실제로 이메일을 보내는 대신에 브라우저에서 이메일을 볼 수 있도록 해 준다.

셋업을 하기 위해서는 `config/environments/development.rb` 파일을 열고 아래와 같이 추가한다.

{%ace edit=false, lang='ruby', theme='monokai'%}
config.action_mailer.delivery_method = :letter_opener
{%endace%}

이제 회원 등록시 인증을 위한 이메일 브라우저에서 확인할 수 있을 것이다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/FoundBlog/2014-06-10_11-22-34_zps5afcd5bc.png)


---

> **소스보기** https://github.com/luciuschoi/foundblog_app/tree/제02장6절

---

_**References:**_

1. [ryanb/letter_opener](https://github.com/ryanb/letter_opener)
