---
layout: post
title: NPM Private Repository 만들기
date: 2011-06-21 23:40:50.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
tags:
- JavaScript
- Node.js
- npmjs
---
NPM Repository 만들기는 생각보다 쉽습니다.

사실, 이 방법은 Repository를 만드는거라기 보다 http://www.npmjs.org를 Cloning 하는거입죠.

https://github.com/isaacs/npmjs.org 에 가면 어떻게 하는지 잘 나와있는데, 일단 CouchDB가 설치되어 있어야 합니다.

CouchDB가 설치되어 있다는 가정하에 몇가지 작업이면 뚝딱 만들 수 있습니다

>'왜 Repository를 따로 두느냐?'

라는 질문을 하신다면...

>'Maven도 Private Repository를 따로 두는것과 같은 이유입니다.'

라고 답하고 싶습니다.

일단 과정은 간단합니다. 따라해볼까요?

1. 설치한 CouchDB에 **registry** 라는 DB를 생성합니다.
2. npmjs.org를 Cloning 합니다.
```sh
$ git clone https://github.com/isaacs/npmjs.org.git
$ cd npmjs.org
```
3. CouchApp을 설치합니다.
```sh
$ sudo npm install couchapp -g
$ npm install couchapp
```
4. 아래 명령을 실행해 registry와 search를 Sync 시킵니다.
```sh
$ couchapp push registry/app.js http://localhost:5984/registry
$ couchapp push www/app.js http://localhost:5984/registry ## 검색 UI와 관련. 필요없음 안해도 된다.
```
5. 아래 명령을 실행해 NPM Repo를 복제합니다. (가장 오래걸림. 크기가 2.93GB 정도 되는듯)
```sh
$ curl -X POST -H "Content-Type:application/json" http://localhost:5984/_replicate -d '{"source":"http://isaacs.couchone.com/registry/", "target":"registry"}'
```
6. NPM Client에 새로만든 Repo를 등록시킵니다. 그런데 등록도 아래 3가지 방법이 있더군요.(등록 2, 바로 사용 1)
6.1. `~/.npmrc` 를 만들거나,
```
registry = http://localhost:5984/registry/_design/app/_rewrite
```
6.2. 바로 설정한다.
```sh
$ npm config set registry http://localhost:5984/registry/_design/app/_rewrite
```
6.3. 필요할 때만 쓴다.
```sh
$ npm --registry http://localhost:5984/registry/_design/app/_rewrite install <package>
```

사실... 이 포스트는 https://github.com/isaacs/npmjs.org 내용 그대로 읽어다 붙인거 밖에 안됩니다. 아래 5번 Step 하다가 좀 지루해서 써본거라는..

**실제 적용해보면서 쉽게 안되던 부분을 정리해 본다.**

**덧1 )**
 CouchDB Configuration에서 아래 해당하는걸 찾아 true에서 false로 바꿔줄것. (insecure_rewrite_rule 관련 에러 피해감)

**secure_rewrites** false

**덧2 )**
NPM Repository를 `http://localhost:5984/registry/_design/app/_rewrite` 와 같이 쓸 경우 NPM에서 제대로 못읽는다.

`/etc/couchdb/local.ini` 또는 `/usr/local/etc/couchdb/local.ini` 부분을 다음과 같이 수정하면 NPM 쓰기가 편해진다.

```
[vhosts]
localhost:5984 = /registry/_design/app/_rewrite
``` 
