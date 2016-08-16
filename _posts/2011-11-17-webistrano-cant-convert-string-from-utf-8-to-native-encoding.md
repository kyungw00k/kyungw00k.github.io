---
layout: post
title: "[Webistrano] Can't convert string from 'UTF-8' to native encoding"
date: 2011-11-17 21:31:53.000000000 +09:00
type: post
published: true
status: publish
categories:
- Webistrano
---
웹 좀 뒤져보다 보면 SVN 사용시에 위 메세지 처리와 관련해서 주로 아래와 같이 하라고 가이드 하고 있다.

```sh
export LC_CTYPE=en_US.UTF8
```

하지만 위를 서버에서 콘솔로 들어가 입력한다고 Webistrano에서 바로 받아먹을 수는 없는 노릇인지라...

`[webistrano 설치 경로]/lib/webistrano/template/base.rb` 에 한줄 추가하는것으로 해결할 수 있다.

```ruby
module Webistrano

# ...

 TASKS = <<-'EOS'
 # allocate a pty by default as some systems have problems without
 default_run_options[:pty] = true
 default_environment["LC_CTYPE"] = "ko_KR.UTF-8" # System Locale이 ko_KR.UTF-8임. 이 부분을 입맛에 맞게 고치면 된다.

# ...
```

Rails도 한지 오래되서 그런지 소스 까볼때마다 새롭다. =_=;;
