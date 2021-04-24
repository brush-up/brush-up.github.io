---
title:  "[iOS] ios9 beta3에서의 문제"
excerpt: "ios9 beta3버전에서 발생했던 문제."

categories:
  - ios
tags:
  - [iOS, Programming]

toc: true
toc_sticky: true
 
date: 2015-07-23
last_modified_at: 2015-07-23
---
# 회고
* 10년이 지난 지금 이게 무슨 필요가 있냐 싶지만.... 일단 글을 옮기는  목적에만 충실히 하기 위해...

# 내용

ios9 beta3 부터 client가 wss 로 server에 접근시 서명알고리즘이 sha1이면 문제가 되는듯하다..

(CFNetwork SSLHandshake failed(-9850) 에러 발생)

-->

인줄알았으나 더 확인해보니

OpenJDK 7에서 기본적으로 지원하는 프로토콜 문제였음

https://bugs.launchpad.net/ubuntu/+source/openjdk-7/+bug/1314113

ios9과 OpenJDK 7은 안맞는거로.. 

```
OPENJDK 7은

ProtocolTest
Supported Protocols: 5
 SSLv2Hello
 SSLv3
 TLSv1
 TLSv1.1
 TLSv1.2
Enabled Protocols: 2
 SSLv3
 TLSv1
```

```
OPENJDK 8은
 Supported Protocols: 5
 SSLv2Hello
 SSLv3
 TLSv1
 TLSv1.1
 TLSv1.2
Enabled Protocols: 3
 TLSv1
 TLSv1.1
 TLSv1.2
```


이하 문제해결하면서 찾아보았던 참고자료들..

How to generate x509 SHA256 hash self-signed certificate using OpenSSL
http://techglimpse.com/sha256-hash-certificate-openssl/

http://docs.oracle.com/cd/E19900-01/820-0849/ablra/index.html

인증서 생성 샘플

C:\Program Files\Java\jdk1.8.0_45\bin> keytool -genkey -keystore keystore.jks -keyalg RSA -sigalg SHA256withRSA

인증서 내용 보기 샘플

C:\Program Files\Java\jdk1.8.0_45\bin> keytool -list -v -keystore keystore.jks


https://developer.apple.com/videos/wwdc/2015/?id=703

https://developer.apple.com/library/prerelease/ios/technotes/App-Transport-Security-Technote/index.html#//apple_ref/doc/uid/TP40016240 

https://developer.apple.com/videos/wwdc/2015/?id=706

https://bugs.launchpad.net/ubuntu/+source/openjdk-7/+bug/1314113



추가작성1 (2015.08.13)

ios9 beta3부터 beta4까지안되더니 오늘 beta5 부터는 OpenJDK7에서도 정상적으로 동작을 한다. 

- 아래는 그때당시 릴리즈 노트

https://developer.apple.com/library/prerelease/ios/releasenotes/General/RN-iOSSDK-9.0/




```
Networking

Note

When negotiating a TLS/SSL connection with Diffie-Hellman key exchange, iOS 9 requires a 1024-bit group or larger. These connections include:
	Secure Web (HTTPS)
	Enterprise Wi-Fi (802.1X)
	Secure e-mail (IMAP, POP, SMTP)
	Printing servers (IPPS)

DHE_RSA cipher suites are now disabled by default in Secure Transport for TLS clients. This may cause failure to connect to TLS servers that only support DHE_RSA cipher suites. Apps that explicitly enable cipher suites using SSLSetEnabledCiphers are not affected and will still use DHE_RSA cipher suites if explicitly enabled.

Safari may see a “Safari can’t establish a secure connection to the server” error page. Safari and other clients of CFNetwork API (NSURLSession, NSURLConnection, CFHTTPStream, CFSocketStream, and Cocoa equivalents) display the “CFNetwork SSLHandshake failed” error in Console.
```