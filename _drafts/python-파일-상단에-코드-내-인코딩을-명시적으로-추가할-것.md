---
layout: post
title: 파일 상단에 코드 내 인코딩을 명시적으로 추가할 것
date: 2016-04-08 11:01:49.000000000 +09:00
type: post
categories:
- Python
---
## Problem
```
SyntaxError: Non-ASCII character '\xec' in file <<file>> on line XX, but no encoding declared;
see http://python.org/dev/peps/pep-0263/ for details  
```

## Solution
```py
# -*- coding: utf-8 -*-
```

파일 상단에 위 주석을 추가한다.

## Reference
http://python.org/dev/peps/pep-0263/
