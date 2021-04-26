---
title:  "웹브라우저에서 https 통신 할때(인증서 관련) flow"
excerpt: "웹브라우저에서 https 통신 할때(인증서 관련) flow"

categories:
  - etc
tags:
  - [etc]

toc: true
toc_sticky: true
 
date: 2020-07-30
last_modified_at: 2020-07-30

published: true
---

* 기본적으로 AES, RSA에 대한 상식은 알고있다고 가정
* 일반적으로 알고있는 https 통신 시퀀스
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2020-07-30-etc-https-flow-001.png)
* 출처 : https://www.researchgate.net/figure/HTTPS-message-sequence-diagram-with-detailed-TLS-handshaking-steps_fig1_306187575
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2020-07-30-etc-https-flow-002.png)

* ```인증서 관련 내용```
    * 1. 인터넷 사이트에서 `사이트정보` 와 `사이트 공개키` 를 인증 기관에 제출
    * 2. 인증 기관에는 제출받은 `사이트정보` 와 `사이트 공개키` 를 ```인증기관의 개인키```로 암호화  : 이것이 사이트 인증서임
    * 3. 인증기관은 웹브자우저에게 인증기관의 공개키를 제공에서 심게하고, 사이트에는 사이트 인증서를 전달.
    * 4. 사용자가 웹브라우저로 사이트에 접속하면 사이트는 자신의 인증서를 웹 브라우저에 내려줌
        * ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2020-07-30-etc-https-flow-003.png) (이부분 맞나?)
    * 5. 웹 브라우저는 내장된 인증기관의 공개키로 사이트의 인증서를 복호화 후 검증함(사이트 정보와 사이트 공개키 추출)
    * 6. 추출한 공개키를 가지고 생성한 대칭키를 암호화해서 사이트에 전달
        * ![image]({{ site.url }}{{ site.baseurl }}/assets/images/2020-07-30-etc-https-flow-004.png) (이부분 맞나?)
    * 7. 사이트는 자신의 개인키로 복호화해서 대칭키 획득. 이후는 대칭키로 동작.



* 더 확인해보자
    * 인증서 관련
        * http://blog.securekim.com/2015/12/blog-post.html
        * http://blog.naver.com/alice_k106/221468341565

* 참고
    * https://opentutorials.org/course/228/4894