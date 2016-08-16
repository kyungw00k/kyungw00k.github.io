---
layout: post
title: "Error: ansible requires a json module, none found!"
date: 2016-04-21 10:10:50.000000000 +09:00
type: post
categories:
- Ansible
---

구 버전 OS(ex. Centos 5)를 대상으로 Ansible을 사용하는 경우에 이러한 에러가 발생한다.

해당 머신에 `python-simplejson` 패키지를 설치하면 해결!
