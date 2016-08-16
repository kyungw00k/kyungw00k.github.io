---
layout: post
title: GitlabHQ + LDAP
date: 2011-11-28 23:04:26.000000000 +09:00
type: post
published: true
status: publish
categories:
- Tooling
tags:
- git
- gitlabhq
- ldap
---
![](/images/2011/11/28/gitlab.png)

근데 시작에 앞서... GitlabHQ가 뭘까?

>나는 Free Git Repository Management App임

이해하기 쉽게 설명하자면 일단 Github은 아니지만, Github스러운 Git Management App이라고 할 수 있다.

크게 다른점이 하나 있다면...

Github은 기본적으로 Repository가 Public으로 read only이지만, GitlabHQ에서 Repository는 기본적으로 Private이다.

이 포스트를 쓰는 시점에
[GitlabHQ Release](http://gitlabhq.com/releases.html)는 Pronto(v1.2)까지 되었고, Moderno(v2.0)부터는 Gitolite를 공식 지원한다고 한다.

LDAP는 공식적으로 GitlabHQ에서 지원하지 않지만..
[붙이는데 그닥 어렵지 않다는 Issue Comment](https://github.com/gitlabhq/gitlabhq/issues/42)를 보고, 붙여보았다.

LDAP 붙이기 위해
- [omniauth-ldap](https://github.com/pantsu/gitlabhq/commits/ldap_and_db_auth)와
- [devise_ldap_authenticatable](https://github.com/cschiewek/devise_ldap_authenticatable)

둘 다 써봤지만, 써보고 내린 결론은 [devise_ldap_authenticatable](https://github.com/cschiewek/devise_ldap_authenticatable)이 가장 간단하고 빠르고 쉽게 설치할 수 있고, **무엇보다 정신건강에 이롭다.**

요걸로 붙여보자.

1. GitlabHQ를 Checkout 받는다.
```sh
$ git clone https://github.com/gitlabhq/gitlabhq.git
$ cd gitlabhq/
```

2. Gemfile을 수정해 devise 바로 아래에 `gem 'devise_ldap_authenticatable'`을 추가한다.
```ruby
source 'http://rubygems.org'

gem 'rails', '3.1.1'

gem 'sqlite3'
gem 'devise', "1.5.0"
gem 'devise_ldap_authenticatable'
gem 'stamp'
gem 'kaminari'
# ...
```

3. bundle install/update를 실행한다.
```
$ bundle install
Updating http://github.com/gitlabhq/grit.git
Updating http://github.com/ctran/annotate_models.git
Fetching source index for http://rubygems.org/
...
Using thin (1.3.1)
Using turn (0.8.3)
Using uglifier (1.1.0)

Your bundle is complete! Use `bundle show [gemname]` to see where a bundled gem is installed.
```

gem 설치가 완료되었으니, `devise_ldap_authenticatable`를 적용시켜보자.

적용시키는 방법은 무척 간단하다.

devise_ldap_authenticatable github 위키에 나와있는 설치법은 다음과 같다.

```sh
$ rails generate devise:install
$ rails generate devise MODEL_NAME
$ rails generate devise_ldap_authenticatable:install
```

하지만 GitlabHQ는 이미 devise가 설치되어 있으므로, 마지막 단계만 실행해주면 되겠다.

4. devise_ldap_authenticatable 설치
```sh
$ rails generate devise_ldap_authenticatable:install
      create  config/ldap.yml
      insert  config/initializers/devise.rb
        gsub  app/models/user.rb

insert  app/controllers/application_controller.rb
```

이제 남은건, LDAP 설정을 추가하고, `initializer/devise.rb`를 수정하면 되겠다. LDAP 설정은 서버마다 다르므로, 일단 `devise.rb`부터 수정해 보자.

5. `config/initializers/devise.rb` 수정
```ruby
# Use this hook to configure devise mailer, warden hooks and so forth. The first
# four configuration values can also be set straight in your models.
Devise.setup do |config|
  # ==> LDAP Configuration
  # config.ldap_logger = true                             # 로그 찍기
  # config.ldap_create_user = false                       # true로 바꿔주면 로그인시 계정이 없으면 생성된다.
  # config.ldap_update_password = true
  # config.ldap_config = "#{Rails.root}/config/ldap.yml"  # 생성된 파일을 사용한다.
  # config.ldap_check_group_membership = false
  # config.ldap_check_attributes = false
  # config.ldap_use_admin_to_bind = false                 # 인증시 ldap.yml에 설정한 admin 계정을 이용한다.
  # config.ldap_ad_group_check = false

  # ==> Mailer Configuration
  #  ...

  config.authentication_keys = [ :login ]
```

위의 주석에서 필요한 부분을 골라 주석을 해제한다.

제 경우는...

```ruby
config.ldap_logger = true # 잘 되는지 log을 찍어봤고
config.ldap_config = "#{Rails.root}/config/ldap.yml" # config/ldap.yml을 사용했고,
config.ldap_create_user = true # 로그인시 계정이 없으면 자동으로 생성하게 했고,
config.ldap_use_admin_to_bind = true # mail 정보를 검색하기 위해 먼저 Admin 계정으로 로그인했다.
```

이제 남은건 ldap 서버 정보를 ldap.yml에 채워넣으면 되겠다.


이때 주의할 점은 GitlabHQ 로그인 시 메일 주소를 넣고 있기 때문에, 사용자 인증과 관련해서 Attribute를 mail로 바꿔주기만 하면 된다.

6. `config/ldap.yml` 수정

```yaml
## Authorizations
# Uncomment out the merging for each enviornment that you'd like to include.
# You can also just copy and paste the tree (do not include the "authorizations") to each
# enviornment if you need something different per enviornment.
authorizations: &AUTHORIZATIONS
  group_base: ou=groups,dc=test,dc=com
  ## Requires config.ldap_check_group_membership in devise.rb be true
  # Can have multiple values, must match all to be authorized
  required_groups:
    # If only a group name is given, membership will be checked against "uniqueMember"
    - cn=admins,ou=groups,dc=test,dc=com
    - cn=users,ou=groups,dc=test,dc=com
    # If an array is given, the first element will be the attribute to check against, the second the gr>
    - ["moreMembers", "cn=users,ou=groups,dc=test,dc=com"]
  ## Requires config.ldap_check_attributes in devise.rb to be true
  ## Can have multiple attributes and values, must match all to be authorized
  require_attribute:
    objectClass: inetOrgPerson
    authorizationRole: postsAdmin

## Enviornments

development:
  host: localhost
  port: 389
  attribute: cn  # 인증에 사용할 Attribute. GitlabHQ는 mail을 사용하므로, mail을 쓰면 된다.
  base: ou=people,dc=test,dc=com
  admin_user: cn=admin,dc=test,dc=com
  admin_password: admin_password
  ssl: false
  # <<: *AUTHORIZATIONS

test:
  host: localhost
  port: 3389
  attribute: cn
  base: ou=people,dc=test,dc=com
  admin_user: cn=admin,dc=test,dc=com
  admin_password: admin_password
  ssl: simple_tls
  # <<: *AUTHORIZATIONS

production:
  host: localhost
  port: 636
  attribute: cn
  base: ou=people,dc=test,dc=com
  admin_user: cn=admin,dc=test,dc=com
  admin_password: admin_password
  ssl: start_tls
  # <<: *AUTHORIZATIONS
```

이미 Ruby와 Rails를 잘 아시는 분들이라면 별 무리 없이 설치했겠지만, 초반에 omniauth로 삽질 했더니 좀 오래걸렸다. =_=;


아무쪼록 GitlabHQ에서 LDAP로 로그인이 필요하신 분들에게 도움이 되었음 좋겠다.
