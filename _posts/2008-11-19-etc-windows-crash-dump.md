---
title:  "[etc]Windows crash dump"
excerpt: "윈도우에서 crash dump를 생성하도록 해보자."

categories:
  - etc
tags:
  - [etc, Windows]

toc: true
toc_sticky: true
 
date: 2008-11-19
last_modified_at: 2018-11-19
---

# 회고
* 10년이상 지난 지금 이게 무슨 필요가 있냐 싶지만.... 일단 글을 옮기는  목적에만 충실히 하기 위해... 갈수록 편해지는 세상

# 내용
## 크래쉬 덤프
* drwtsn32
	* 첨부한 파일로 레지스터를 바꾸면 크래쉬시 drwtsn32이 자동으로 덤프파일을 남긴다.
	* Dr. Watson 실행화면
		![image]({{ site.url }}{{ site.baseurl }}/assets/images/2008-11-19-etc-windows-crash-dump-001.jpg)
	* default로 덤프파일이 생성되는 곳은 다음과 같다.
	```
	C:\Documents and Settings\All Users\Application Data\Microsoft\Dr Watson\user.dmp
	```
	* 크래쉬 상황시 덤프파일을 남기는 상황
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2008-11-19-etc-windows-crash-dump-002.jpg)
	* 덤프파일과 함께 크래쉬 상황이 들어가있는 파일도 함께 생성된다
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2008-11-19-etc-windows-crash-dump-003.jpg)

* ntsd
	* 첨부한 파일로 레지스터를 수정하면 크래쉬상황이 발생시 ntsd가 자동으로 덤프파일을 남긴다.
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2008-11-19-etc-windows-crash-dump-004.jpg)

* vsjitdebugger.reg : 이아이로도 만들수 있는데 굳이 이걸 쓸 이유는 ....


## 유저덤프.
Userdump.exe : 이 아이를 이용해서 유저덤프를 만들수가 있다.
http://support.microsoft.com/default.aspx?scid=kb%3Bko%3B241215   : 여기를 참조하면 도움이 될듯하지만..

# 첨부파일이었던 내용
* drwtsn32.reg
```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AeDebug]
"Auto"="1"
"Debugger"="drwtsn32 -p %ld -e %ld -g"
"UserDebuggerHotKey"=dword:00000000

```

* ntsd.reg
```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AeDebug]
"Auto"="1"
"Debugger"="ntsd.exe -p %ld -e %ld -g -c \".dump /m /a /u C:\\crash.dmp;q\""
"UserDebuggerHotKey"=dword:00000000
```

* vsjitdebugger.reg
```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AeDebug]
"Auto"="0"
"Debugger"="\"C:\\WINDOWS\\system32\\vsjitdebugger.exe\" -p %ld -e %ld"
"UserDebuggerHotKey"=dword:00000000
```
* Userdump.exe
	* 굳이 이것까진 안옮김