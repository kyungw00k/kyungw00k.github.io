---
layout: post
title: textarea ng-model is not updated in IE with CJK IME
date: 2016-02-22 11:40:38.000000000 +09:00
type: post
categories:
- AngularJS
---
## Issue
https://github.com/angular/angular.js/issues/6656

## Problem
compositionstart, compositionend 이벤트를 이용하여 조합이 된 값에 대해서만 모델에 바인딩하는데, compositionend 이벤트가 잘 호출되지 않아 문제 발생

## Solution
compositionstart, compositionend 관련부분을 주석처리
