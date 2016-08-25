---
layout: post
title: GoAccess
date: 2016-01-26 15:51:20.000000000 +09:00
type: post
categories:
- logging
---

> an open source real-time web log analyzer and interactive viewer that runs in a terminal in \*nix systems (from [GoAccess](http://goaccess.io/))

## Install

### Mac
```sh
brew install goaccess
```

## Usage for nginx `access.log`
```sh
`which goaccess` -f /var/log/nginx/access.log -a -g -H -M --real-os --log-format='%h %^[%d:%t %^] "%r" %s %b "%R" "%u"' --date-format="%d/%b/%Y" --time-format='%H:%M:%S' > report.html
```
