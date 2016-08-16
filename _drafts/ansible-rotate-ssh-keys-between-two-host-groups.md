---
layout: post
title: Rotate SSH Keys between two host groups
date: 2016-05-04 16:04:18.000000000 +09:00
type: post
categories:
- Ansible
---

SSH로 Remote 서버들을 관리할 때 느끼는거지만..

- yes/no 묻지 않게 known_host 자동 추가
- authorized_key에 허용 서버 키 추가

요거 꼭 하고 싶었다. 그래서 playbook으로 만들어봤다.

{% gist kyungw00k/d56547b1e30b5d72316e7df0e9d99f92 %}
