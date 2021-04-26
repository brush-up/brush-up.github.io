---
title:  "Windows 에서 Powershell 파워쉘로 netstat 결과를 http 요청 보내는 샘플"
excerpt: "Windows 에서 Powershell 파워쉘로 netstat 결과를 http 요청 보내는 샘플"

categories:
  - etc
tags:
  - [etc, windows]

toc: true
toc_sticky: true
 
date: 2021-04-26
last_modified_at: 2021-04-26

published: true
---

# intro
* windows 장비에서 CLOSE_WAIT 나 TIME_WAIT 인 소켓의 갯수를 모니터닝 할 일이 발생함
* https://docs.microsoft.com/ko-kr/powershell/ 파워쉘을 이용하여 해보기로함
* 이하 작업결과물을 샘플용도로 보관하기 위함

# 내용
### 전체 구조
```
└─작업폴더(C:\Script)
     │  start.bat
     │  netstat.ps1
     ├─Util
     │  printmsg.ps1
     │  webhook.ps1
     └─Log
```

### 각 파일의 내용
* .start.bat
    * 시작점
```ps
@echo on
cd C:\Script

powershell "& { Set-ExecutionPolicy RemoteSigned } "
powershell .\netstat_test.ps1
powershell "& { Set-ExecutionPolicy Restricted } "
```

* netstat.ps1
    * WAIT_MAX_COUNT 보다 큰 숫자일 경우 잘 잘 처리하기 위해서.
```ps

. .\Util\printmsg.ps1
. .\Util\sendmail.ps1

## -------------------------------------------------------------------------------------

PrintMsg "Script Start [netstat_check]"

## -------------------------------------------------------------------------------------

$WAIT_MAX_COUNT = 1000
$select_count = (netstat -an | Select-String "WAIT").count

if ($select_count -gt $WAIT_MAX_COUNT ) {
   $msg   = "ERROR netstat WAIT count[$select_count]"
   $title = "[netstat] bigger than " + $WAIT_MAX_COUNT
   PrintMsg $msg
   WebHook $title $msg
} else {
   PrintMsg "netstat WAIT. count[$select_count]"
}
PrintMsg "Script END"

exit
```

* printmsg.ps1
    * 특정위치에 로깅을 위한 용도.
```ps
$log_folder = "C:\Script\Log"
$today = (Get-Date).AddDays(0).ToString("yyyyMMdd")

Function PrintMsg ([string] $msg) {

	$now = Get-Date -Format "yyyy/MM/dd HH:mm:ss"
	$message = "$now $msg"
	Write-Host $message
	$log_path = ${log_folder}+"\netstat_"+ ${today} + ".log"

	if (-not (Test-Path $log_folder -PathType Container)) {
		$temp = mkdir $log_folder
	}
	# 
	Write-Output $message | Out-File ${log_path} -Append -Encoding Default
}

```

* webhook.ps1
    * 특정 url에 post방식으로 body에 특정 json 데이터를 보낸뒤 결과값을 출력하기 위한 용도
```ps
$hostname = HOSTNAME
$URL = "https://.....";

function WebHook ([string] $title, [string] $msg){
	$params = @{
		body = @{
			botName = "MyBot"
			text = "[" + $hostname + "] " +  $title + " : " + $msg
		} | ConvertTo-Json

		uri = $URL
		headers = @{ "Content-Type" = "application/json"}
		method = "Post"
	}

	$response = Invoke-WebRequest @params
	Write-Host $response
}
```

# windows 작업 스케줄러 
* 매일 10분마다 특정 작업을 실행시키기
    * 위의 start.bat를 실행하자.
* 실행방법 2가지
    * 시작 > 모든프로그램 > 보조 프로그램 > 작업 스케줄러 
    * taskschd.msc

* step1
	* ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2021-04-26-etc-windows-powershell-001.png)
* step2
	* ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2021-04-26-etc-windows-powershell-002.png)
* step3
	* ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2021-04-26-etc-windows-powershell-003.png)
* step4
	* ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2021-04-26-etc-windows-powershell-004.png)
* step5
	* ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2021-04-26-etc-windows-powershell-005.png)
* step6
	* ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2021-04-26-etc-windows-powershell-006.png)
