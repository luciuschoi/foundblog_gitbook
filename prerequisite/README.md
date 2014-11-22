# 사전준비 작업

이 책의 내용을 따라하라 하기 위해서는 [<<초보자를 위한 레일스 가이드북>>](https://www.gitbook.io/book/rorlab/railsguidebook) 내용 중의 ["환경준비"](http://rorlab.gitbooks.io/railsguidebook/contents/environments.html)와 ["레일스 설치"](http://rorlab.gitbooks.io/railsguidebook/contents/rails/install.html) 부분을 읽어 보기 바란다.

위의 내용을 참고하여 현재 본인의 로컬 머신에 레일스 프로젝트를 개발하기 위한 제반 환경이 완료된 것으로 간주하고 시작하기로 한다.

## 샘플 프로젝트 'FoundBlog'의 생성

이 책에서 설명과 함께 작성할 레일스 프로젝트의 이름은 [`Foundation 5`](http://foundation.zurb.com)를 사용하여 간단한 블로그를 작성할 것이기 때문에 `'FoundBlog'`라고 명명하기로 한다.

이제 아래와 같이 레일스 프로젝트를 생성하고 프로젝트 디렉토리로 이동한다.

```bash
$ rails new FoundBlog && cd FoundBlog
```

레일스 4.1부터는 레일스 어플리케이션 프리로더(preloader)인 [`spring`](https://github.com/rails/spring)을 디폴트로 포함하고 있다. `spring`은 어플리케이션을 백그라운드에서 실행하기 때문에 개발 속도를 향상시켜 준다. 프로젝트 디렉토리 내에서 `spring` 서버의 시작과 종료는 자동으로 실행되기 때문에 개발자가 신경 쓸 필요가 없다. 따라서 `spring`을 이용한 빠른 실행을 원하다면 `bin/rake` 또는 `bin/rails`와 같이 명령 앞에 `bin/` 경로를 추가하여 실행하면 된다.

```bash
$ spring status
Spring is not running.
```

위와 같이 현재 `spring` 서버가 시작되지 않은 상태라 하더라고 `bin` 디레토리에 위치한 실행명령(rake, rails 등)을 사용하면 자동으로 `spring` 서버가 실행되는 것을 확인할 수 있다.

위의 명령을 최초로 실행시에는 다소 시간이 걸리더라도 이후부터는 `spring` 서버 덕분에(어플리케이션이 백그라운드에서 실행되고 있기 때문) 매우 빠르게 실행된다.

```bash
$ bin/rake about
About your application's environment
Ruby version              2.1.2-p95 (x86_64-darwin13.0)
RubyGems version          2.2.2
Rack version              1.5
Rails version             4.1.1
JavaScript Runtime        Node.js (V8)
Active Record version     4.1.1
Action Pack version       4.1.1
Action View version       4.1.1
Action Mailer version     4.1.1
Active Support version    4.1.1
Middleware                Rack::Sendfile, ActionDispatch::Static, Rack::Lock, #<ActiveSupport::Cache::Strategy::LocalCache::Middleware:0x007f9a91bca2b8>, Rack::Runtime, Rack::MethodOverride, ActionDispatch::RequestId, Rails::Rack::Logger, ActionDispatch::ShowExceptions, ActionDispatch::DebugExceptions, ActionDispatch::RemoteIp, ActionDispatch::Reloader, ActionDispatch::Callbacks, ActiveRecord::Migration::CheckPending, ActiveRecord::ConnectionAdapters::ConnectionManagement, ActiveRecord::QueryCache, ActionDispatch::Cookies, ActionDispatch::Session::CookieStore, ActionDispatch::Flash, ActionDispatch::ParamsParser, Rack::Head, Rack::ConditionalGet, Rack::ETag
Application root          /Users/hyo/prj/r4/FoundBlog
Environment               development
Database adapter          sqlite3
Database schema version   0
```

그리고 다시 `spring` 서버의 상태를 확인하자.

```bash
$ spring status
Spring is running:

 7649 spring server | FoundBlog | started 9 secs ago
 7650 spring app    | FoundBlog | started 9 secs ago | development mode
```

이제는 `spring server`가 실행 중에 있는 것을 확인할 수 있다.

---

> **소스보기** https://github.com/LuciusChoi/foundblog/tree/제1장

