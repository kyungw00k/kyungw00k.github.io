---
layout: post
title: python setup.py install 로 설치한 파일들 삭제하기
date: 2011-11-25 14:52:52.000000000 +09:00
type: post
published: true
status: publish
categories:
- Python
---
**1. 다시 설치하는데 --record 옵션을 줘서 기록한다.**

```sh
$ python setup.py install --record uninstall.txt
```

**2. 설치한 파일을 하나씩 삭제한다.**

```
cat uninstall.txt | xargs rm -rf
```

(Reference : [How do I uninstall gitosis](http://serverfault.com/questions/91907/how-do-i-uninstall-gitosis))
