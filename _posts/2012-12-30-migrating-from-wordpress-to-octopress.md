---
layout: post
title: Migrating From Wordpress to Octopress
date: 2012-12-30 17:35:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Tutorial
tags:
- blog
- markdown
---

Markdown으로 모든 블로그 Post를 이전해야 해서, 기존에 만들어진 것들이 어떻게 돌아가는지 확인도 할겸 Octopress로 옮겨보았다(원래는 Haroopress로 이전하려고 했지만…).

### Data Migration Tool

#### [exitwp](https://www.google.co.kr/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&ved=0CCwQFjAA&url=https%3A%2F%2Fgithub.com%2Fthomasf%2Fexitwp&ei=YvTgUPmNAsSYiAeSyICYCA&usg=AFQjCNFyTaO04GcC_kacLBW75YwHUXaHTg&sig2=f3NlfBCtHh73bzNtRgdacQ&bvm=bv.1355534169,d.aGc)

* 대표적으로 많이 사용하는 Python Script
* 첨부파일까지 다운로드 해주는 기능이 있긴 하지만 포스트 링크까지 교체해주지는 않음.
* Markdown 결과물이 그리 좋아 보이지 않음

### [Wordpress To Octopress by @reiot](http://reiot.com/2011/09/21/wordpress-to-octopress/)

* 한글 파일명으로 잘 보임(글 제목이 CJK인 경우 별도 처리)
* 순수 Markdown 문법으로 변환되는건 아니고, 부분적으로 Octopress Template 코드를 사용함.

결과적으로 [@reiot](http://twitter.com/reiot)님이 쓰신 포스트를 사용해서 Markdown으로 손쉽게 이전하였다.

### Theme
> 좀 더 아름답게 써볼 수 없을까?


*[Octopress Themes](http://octopressthemes.com/) - 2.1 호환 안되는것 같음
*[3rd Party Octopress Themes](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)

이것 저것 다 써봤지만…

일단 현재 적용된 Theme는
[greyshade](https://github.com/shashankmehta/greyshade)다.

### TODO

>글 쓰는 Process를 매끄럽게 할 수는 없나?

* [Synced and scheduled blogging with Octopress](http://instant-thinking.de/2012/08/03/synced-and-scheduled-blogging-with-octopress/)
* [Remote Blogging With Octopress](http://www.candlerblog.com/2012/04/01/remote-octopress-workflow/)
