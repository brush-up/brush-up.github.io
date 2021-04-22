---
title:  "[iOS] symbolicatecrash 사용하기"
excerpt: "symbolicatecrash 를 이용해  crash report를 보자"

categories:
  - ios
tags:
  - [iOS, Programming]

toc: true
toc_sticky: true
 
date: 2014-10-11
last_modified_at: 2014-10-11
---
# 회고
* 10년이 지난 지금 이게 무슨 필요가 있냐 싶지만.... 일단 글을 옮기는  목적에만 충실히 하기 위해...

# 내용

* 순서대로 실행
* xcode6 기준

```
1. alias symbolicate="/Applications/Xcode.app/Contents/SharedFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/symbolicatecrash -v"
2. export DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer"
3. symbolicate -o "xxx.crash" "MyAppName.app"
```
