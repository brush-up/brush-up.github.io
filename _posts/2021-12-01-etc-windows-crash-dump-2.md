---
title:  "[etc]Windows crash dump 2"
excerpt: "윈도우에서 crash dump를 생성하도록 해보자. 2"

categories:
  - etc
tags:
  - [etc, Windows]

toc: true
toc_sticky: true
 
date: 2021-12-01
last_modified_at: 2021-12-01
---

# intro
* 이전에 https://brush-up.github.io/etc/etc-windows-crash-dump/ 이런글을 남겼었는데..

# 여튼
예전에는 windows 에서 application에서 crash 가 발생할시 dr. watson 을 통해서 crash dump를 만들수 있었는데
windows7? 이후부터는 dr. waston 이 사라지고 WER을 통해서만 있는듯
기본적으로 WER(Windows Error Reporting) 시스템이 MS에 덤프를 전송하지만 로컬에는 남지 않도록 되어있다고 한다.
(그런데 dump파일이 수십기가고 그거면 어떻게 동작할까?)

아래 MS 개발자센터에서 제공하는 글을 읽고 설정해보자
https://docs.microsoft.com/ko-kr/windows/win32/wer/collecting-user-mode-dumps?redirectedfrom=MSDN

```
레지스터리  HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps 를 바꿔야 한다.
```

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2021-12-01-etc-windows-crash-dump.png)


* 이용하기 쉽게 사용하기위한 usermode_dump_enable.reg 파일 내용. (파일을 만들어서 실행만하면 된다.)

```

Windows Registry Editor Version 5.00 

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps]
"DumpFolder"=hex(2):25,00,4C,00,4F,00,43,00,41,00,4C,00,41,00,50,00,50,00,44,00,41,00,54,00,41,00,25,00,5C,00,43,00,72,00,61,00,73,00,68,00,44,00,75,00,6D,00
"DumpCount"=dword:00000010
"DumpType"=dword:00000002
"CustomDumpFlags"=dword:00000000

```

* 그런데 글들을 읽어보면 WER 서비스를 활성화 해야한다는 말도 있고, 다들 설명이 다름. 아놔.
* `실제로 테스트해볼 필요있음`


# 실습
* 실제로 설정하고 crash dump가 잘 남는지 봤는데, 안되는것 확인, 무엇이 문제지... 고민하다 일단 홀딩


# 참고자료 
[https://wendys.tistory.com/36](https://wendys.tistory.com/36)
[https://helpx.adobe.com/kr/xd/kb/how-to-generate-crash-dump-on-windows-machine.html](https://helpx.adobe.com/kr/xd/kb/how-to-generate-crash-dump-on-windows-machine.html)
[http://forensic-proof.com/archives/4358](http://forensic-proof.com/archives/4358)
[https://ezbeat.tistory.com/459](https://ezbeat.tistory.com/459)

