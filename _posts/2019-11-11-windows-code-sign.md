---
title:  "리눅스 장비에서 windows PE파일 코드사이닝 하기"
excerpt: "리눅스 장비에서 windows PE파일 코드사이닝을 해보자."

categories:
  - etc
tags:
  - [etc, Linux, Windows]

toc: true
toc_sticky: true
 
date: 2019-11-11
last_modified_at: 2019-11-11

published: true
---


# 전자 서명을 위한 사전 지식
* 네이버 애플리케이션의 전자 서명 원리
    * https://d2.naver.com/helloworld/744920
    * 쉬운 개념도는 아래와 같다
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-11-11-windows-code-sign-001.png)
* 추가로 명성치도 중요하게 생각하자
    * https://blog.dramancompany.com/2015/12/처음-windows-설치-파일을-배포하는-개발자들을-위하여/
* PE 파일 어느 부분을 어떻게 건들까?
    * 쉽게 잘 설명되어있는자료는 도저히 못찾겠고 MS에서 배포한 `authenticode_pe.docx` 첨부파일을 읽어보면 좀더 이해할수 있을듯?
        * http://download.microsoft.com/download/9/c/5/9c5b2167-8017-4bae-9fde-d599bac8184a/Authenticode_PE.docx
		![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-11-11-windows-code-sign-002.png)
    * 좀더 심플 버전
        * https://crattack.tistory.com/entry/Code-Sign-분석하기
    * 참고할만한 추가 사이트
        * https://www.comodossl.co.kr/certificate/codesign/guides.aspx
---        
* 참고로 윈도우즈 실행파일이 어떻게 생겼고, 어떻게 동작하는지 궁금하다면 아래책을 추천(웬만한 기술서적은 읽은다음 다  팔아먹어도 이건 안팔고 가지고있음)
    * **Windows 시스템 실행파일의 구조와 원리**
        * http://www.yes24.com/Product/Goods/1493278?scode=032&OzSrank=2


# 인증서 이정도는 알고 가자.
* `그래도 도저히 모르겠다`
    * .pfx는 .p12랑 같아!
```
* 인코딩 (확장자로 쓰이기도 한다.)
	* .der:
		* Distinguished Encoding Representation (DER)
		* 바이너리 DER 형식으로 인코딩된 인증서. 텍스트 편집기에서 열었는데 읽어들일 수 없다면 이 인코딩일 확률이 높다.
	* .pem: X.509 v3 파일의 한 형태
		* PEM (Privacy Enhanced Mail)은 Base64인코딩된 ASCII text file이다.
		* 원래는 secure email에 사용되는 인코딩 포멧이었는데 더이상 email쪽에서는 잘 쓰이지 않고 인증서 또는 키값을 저장하는데 많이 사용된다.
		* -----BEGIN XXX-----, -----END XXX----- 로 묶여있는 text file을 보면 이 형식으로 인코딩 되어있다고 생각하면 된다. (담고있는 내용이 무엇인지에 따라 XXX 위치에 CERTIFICATE, RSA PRIVATE KEY 등의 키워드가 들어있다)
		* 인증서(Certificate = public key), 비밀키(private key), 인증서 발급 요청을 위해 생성하는 CSR (certificate signing request) 등을 저장하는데 사용된다.

* 확장자
	* .crt, .cer 인증서를 나타내는 확장자인 cer과 crt는 거의 동일하다고 생각하면 된다. (cer은 Microsoft 제품군에서 많이 사용되고, crt는 unix, linux 계열에서 많이 사용된다.)
		* 확장자인 cer이나 crt만 가지고는 파일을 열어보기 전에 인코딩이 어떻게되어있는지 판단하긴 힘들다.
	* .key: 개인 또는 공개 PKCS#8 키를 의미
	* .p12: PKCS#12 형식으로 하나 또는 그이상의 certificate(public)과 그에 대응하는 private key를 포함하고 있는 key store 파일이며 패스워드로 암호화 되어있다. 열어서 내용을 확인하려면 패스워드가 필요하다.
	* .pfx: PKCS#12는 Microsoft의 PFX파일을 계승하여 만들어진 포멧이라 pfx와 p12를 구분없이 동일하게 사용하기도 한다.
```
* 참고 url
    * https://www.letmecompile.com/certificate-file-format-extensions-comparison/


# osslsigncode
## Intro
* PE 포맷도 오픈되어있고, 해시방법도 일반적일것이라 생각해서, 리눅스에서 windows PE파일을 건드는 오픈소스가 있을 것 같아 찾아 보니 이런게 있네!
* https://github.com/mtrojnar/osslsigncode
```
* osslsigncode

* WHAT IS IT?
	* osslsigncode is a small tool that implements part of the functionality of the Microsoft tool signtool.exe - more exactly the Authenticode signing and timestamping. But osslsigncode is based on OpenSSL and cURL, and thus should be able to compile on most platforms where these exist.

* WHY?
	* Why not use signtool.exe? Because I don't want to go to a Windows machine every time I need to sign a binary - I can compile and build the binaries using Wine on my Linux machine, but I can't sign them since the signtool.exe makes good use of the CryptoAPI in Windows, and these APIs aren't (yet?) fully implemented in Wine, so the signtool.exe tool would fail. And, so, osslsigncode was born.

* WHAT CAN IT DO?
	* It can sign and timestamp PE (EXE/SYS/DLL/etc), CAB and MSI files. It supports the equivalent of signtool.exe's "-j javasign.dll -jp low", i.e. add a valid signature for a CAB file containing Java files. It supports getting the timestamp through a proxy as well. It also supports signature verification, removal and extraction.
```
* 그런데 전체 옵션에 대한 설명이 그 어디에도 없다. 소스를 보거나  출력된 usage를 보고 짐작을 해야한다.


## 설치 삽질기
### 쉽게 yum으로 설치 해보자.
* osslsigncode
    * sudo yum install osslsigncode
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-11-11-windows-code-sign-003.png)
    * 그런데 버전이 이상하다.
    ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-11-11-windows-code-sign-004.png)
    * 이상해 모르겠다. 지우자.
    ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-11-11-windows-code-sign-005.png)
    * 낮은 버전의 osslsigncode 는 안되는 기능이 너무 많다. 
* openssl
    * 이것도 버전이 상당히 낮다.  
        * 최신버전의 osslsigncode에서는 openssl 1.1.0 이상의 버전이 필요하다.
		![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-11-11-windows-code-sign-006.png)
* yum repository가 막혀있는지, 잘못되었는지, 아니면 우리가 사용중인 OS가 6.X여서 그런것인지 몰라 한참을 고생하다가. 결국.. 소스를 다운받아 빌드를 하기로 결정함 (`리눅스맹`)

### 그래! 설치는 build를 직접 해야 제맛
* 아마 curl, libssh2, gcc, GNU Autotools 는 설치가 되어있을듯
--- 
**openssl**
아래 순서로 설치하자. gcc까지 설치할 일이 없길.(root 권한 필요할지도.)
```
#wget https://www.openssl.org/source/openssl-1.1.0g.tar.gz 
#tar xvfz openssl-1.1.0g.tar.gz
#cd openssl-1.1.0g
#./config --prefix=/usr -fPIC shared
#make
#make install
```
* **wget https://www.openssl.org/source/openssl-1.1.0g.tar.gz**
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-11-11-windows-code-sign-007.png)
* 이하 생략...
* 설치완료
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-11-11-windows-code-sign-008.png)

--- 

**osslsigncode**
https://github.com/mtrojnar/osslsigncode 에서 소스를 다운 받자.
아래 순서로 설치해보자.
```
 ./autogen.sh
  ./configure
  make
  make install
```
* 다시
![image.png](/files/2607208357371186902)
![image.png](/files/2607213392868442433)
![image.png](/files/2607213605816941504)
경로가 꼬임.. 하지만 설치는 됨.
![image.png](/files/2607214096620059812)


* 설치 관련 참고 URL
    * https://ychcom.tistory.com/156



## 이제, 실행 해보자
* 테스트 삼아서 Sample.exe의 인증서 검증해 보기.
![image.png](/files/2608085202800609932)
* 코드사이닝 잘 되는지 한번 해보기
```
./osslsigncode-master/osslsigncode sign -pkcs12 CHAIN_nhn_com.pfx -pass PASSWORD -n yuik_test -h sha256  -t http://timestamp.verisign.com/scripts/timstamp.dll -in Sample.exe -out Sample-signed.exe 
```
![image.png](/files/2608086727664945411)

* 교차 인증서도 적용해보기.
```
./osslsigncode-master/osslsigncode sign -pkcs12 CHAIN_nhn_com.pfx -pass PASSWORD  -ac MSCV_COMODOAddTrust.crt  -n yuik_test -h sha256  -t http://timestamp.verisign.com/scripts/timstamp.dll -in Sample.exe -out Sample-signed-crt.exe 
```
![image.png](/files/2608087538045309880)
* 시간 생성 알고리즘 바꿔보기
```
./osslsigncode-master/osslsigncode sign -pkcs12 CHAIN_nhn_com.pfx -pass PASSWORD  -ac MSCV_COMODOAddTrust.crt  -n yuik_test -h sha256  -t  http://timestamp.comodoca.com/rfc3161  -in Sample.exe -out Sample-signed-crt.exe 
```
![image.png](/files/2608088308218360391)
* 첨부파일 설명 (code sign.zip 압축 풀면 아래와 같이 나온다.)
    * Sample.exe : 원본 파일(코드사인 미적용)
    ![image.png](/files/2608090439412741656)
    * sample_1.exe : 현재 CDA로 생성한 파일
    ![image.png](/files/2608091494706182323)    
    * sample_2.exe: 테스트 삼아 리눅스에서 코드사인 한 파일
    ![image.png](/files/2608092105709704021)
    * sample_3.exe : 교차인증서(MSCV_COMODOAddTrust.crt)추가해서 코드사인 한 파일
    ![image.png](/files/2608093019507661494)


---
* 참고 자료
    * 교차 인증서는 `Windows 커널에 연동되는 시스템 파일`에만 필요 한것 같음(정확하진 않음)
        * https://www.comodossl.co.kr/certificate/codesign/guides.aspx
    * 교차 인증서는 여기서 구할 수 있다.
        * https://www.kicassl.com/codesigncert/instlguid/formInstlGuidList.sg

    ```
    만료된, 즉 유효 기간이 지난 인증서로는 프로그램에 서명할 수 없다. 그러나 인증서가 만료되기 전에 서명한 프로그램은 인증서가 만료된 후에도 유효하다. 단, 이를 위해서는 서명 시 /t 옵션으로 시점 확인 서버(time-stamping server)를 지정해야 한다. CA는 이를 위한 URI를 제공한다.
    다음은 Comodo(또는 Ksoftware)의 시점 확인용 URI를 지정하는 예이다.
    /t http://timesamp.comodoca.com/authenticode
    ```
## TODO
* 듀얼 사이닝 테스트 해보기
* CAB 파일 코드 사이닝 하기

## 기타
* 이 내용 참고하기
    * [게임플랫폼서버팀/117 RE: RE: &#91;한게임&#93; sha256 신규 인증서 관련하여 문의 드립니다.](dooray://1387695619080878080/tasks/2607178472376016725 "closed")
* 리눅스 파일 복사. 
    * 당겨오기
scp irteamsu@hsptst-mvx904:/home1/irteamsu/yuik_test/Sample-signed.exe /home1/yuik/test_CDA/
    * 밀어넣기
    scp ./osslsigncode-master.zip irteamsu@hsptst-mvx904:/home1/irteamsu/yuik_test