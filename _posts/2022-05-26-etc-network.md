---
title:  "[etc]network 기초정리"
excerpt: "network 기초정리"

categories:
  - etc
tags:
  - [etc, Book]

toc: true
toc_sticky: true
 
date: 2022-05-26
last_modified_at: 2022-05-26
---

* 대학시절 구매했던 책도 버릴겸, 어떤 내용이 있었나 한번 훝어 볼겸 겸사 겸사 정리 차원에서
* 해당 내용을 중심으로 조금은 내용 추가해서 정리.

## 프로토콜
### ARP
* 주소 변환 프로토콜(address resolution protocol)
    * IP주소를 물리주소로 변환하기 위해 사용되는 프로토콜
        * 패킷은 물리 조소에 의해 다음 목적지를 찾아가고 IP 주소에 의해 최종 목적지로 찾아간다. 
* ARP 패킷 구조

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-01.png)

* 작동 방식
    * 1. 출발지 시스템에 ARP Cache Table에 목적지 MAC 주소가 있는지 확인한다.
    * 2. MAC 주소가 없으면 ARP request 브로드캐스트로 패킷을 보낸다.
    * 3. ARP request 패킷을 받은 쪽에서 자신의 MAC 주소를 ARP reply 패킷으로 알려준다.
    * 4. ARP reply 패킷을 받은 출발지 시스템은 ARP Cache Table에 해당 정보를 기록한다.

### ICMP
* internet control message protocol 은 네트워크 내에 발생하는 여러가지 문제 제어를 위한 프로토콜이다. 
    * 대표적인 예가 Ping 패킷이다. 

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-02.png)

* ICMP는 크게 가지지로 나뉜다.
    * 오류보고 메시지
        * 라우터나 호스트가 패킷을 처리하는데 있어 문제가 발생할 경우 보내는 메시지
    * 질의 메시지
        * 네트워크 관리자 또는 호스트가 라우터나 다른 호스트에게 현재 상황을 질의할때 사용

### IP 
* internet protocol 
* TCP/IP 프로토콜에서 전송을 담당하는 프로토콜. IP는 신뢰성이 없고 비연결 지향적이며 목적지까지 패킷을 전달하기 위해 최선을 다한다.
    * 전달 과자ㅓㅇ에 문제가 발생할수있다는 것을 가정하고 최선을 다하지만, 결과는 보장하지 못한다. 
* IP 데이터그램 구조

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-03.png)

### TCP
* transmission control protocol
* 신뢰성 있는 통신을 제공하기 위한 프로토콜
* IP는 신뢰성을 제공하지 못하기에 신뢰성을 더하기 위해 IP프로토콜 내에 TCP프로토콜을 함께 실어 보내야 한다.
* TCP 세그먼트 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-04.png)

* 연결지향적이다.
* TCP가 전송하는 데이터는 스트림으로 순차성이 있다. 
* 흐름제어, 오류제어를 수행한다. 


### UDP
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-05.png)
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-06.png)

* 패킷 비교
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-07.png)


## TCP, UDP
* TCP 가 느린 이유 대표적 2가지
    * 연걸 설정 과정과 종료과정
    * 흐름제어
* 결국, 송수신하는 데이터의 양이 큰 경우(유지해야하는 세션이 긴경우) UDP 보다 여러 측면에서 TCP가 효율
* 데이터 양이 작은 경우(유지해야하는 세션이 작고 잦은 연결이 발생하는 경우) UDP가 TCP보다 훨씬 빠르게 동작도 한다.




## TCP 상태 전이 
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-08.png)

* TIME_WAIT 이해하기
    * TIME_WAIT 이란 TCP 상태의 가장 마지막 단계이며, Active Close 즉, 먼저 close()를 요청한 곳에서 최종적으로 남게 되
    * 2*MSL(Maximum Segment Lifetime)동안 유지된다.
    * 필요 이유 정리하기
* CLOSE_WAIT 이해하기
    * Active Close 쪽이 먼저 close()를 수행하고 FIN을 보내면 Passive Close 쪽은 ACK을 보낸 후 어플리케이션의 close()를 수행한다. 
    * 필요 이유 정리하기


## 소켓 옵션
### SO_REUSEADDR
* .소켓 종료를 서버에서 하는냐, 클라에서 하느냐에 따른 차이를 이해하자
    * 클라이언트에서 종료 요청시
        * 서버 재시작시 이슈 없음
    * 서버에서 종료 요청시
        * 서버 재시작시 bind 함수 호출에 이슈 발생 가능성 있음
* 위의 TCP 상태 전이를 보듯
    * 서버측 소켓이 소멸되지 않고 TIME_WAIT 상태에 있을때 발생할 수 있다. 
    * SO_REUSEADDR 옵션이 있으면 TIME_WAIT 상태에 있는 소켓에 할당되어있는 IP, PORT를 새로 시작하는 소켓에 할당해준다.

### TCP_NODELAY
* Nagle 알고리즘
    * 기존에 전송한 패킷이 있을 경우 그 패킷에 대한 ACK를 받아야 다음 전송은 진행하는 알고리즘

### SO_LINGER
* 소켓 닫을때 미전송한 데이터가 있을 경우 지정한 시간만큼 대기하고 소켓을 닫는다.
* 
### SO_KEEPLIVE
* 소켓이 끊어졌는지 감지하지 위한 옵션, 설정하면 연결된 상대에게 주기적(2시간간격)으로 데이터를 보내고 응답을 기다림.
* 응답이 없거나, RST(reset)패킷으로 응다할 경우 소켓연결을 해제함.

### SO_SNDTIMEO, SO_RCVTIMEO
* 블로킹 소켓으로 생성시 송/수신 타임아웃을 설정한다. 




## TCP/IP 네트워크 데이터 전송 과정의 이해
* 이더넷 프레임 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-09.png)
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-10.png)

### ARP 에 의한 물리 주소 변환
* ARP를 통한 물리 주소 확인한다
    * 이렇게 얻어진 물리주소를 가지고 이더넷 프레임을 만들수 있다. 
* ARP 테이블 확인 방법
    * windows
        * arp -a 명령어
    * 리눅스
        * arp 명령어나 cat / proc/net/arp 명령어

### 도메인을 IP로 변환하기
* ARP 테이블를 이용해 기본 게이트웨이의 물리주소를 찾는다.
* 게이트웨이의 물리주소가 ARP테이블에 없다면 ARP패킷을 브로드캐스팅해서 물리주소 변환 과정을 가진다. 
    * 기본 게이트웨이의 물리주소를 알아야만 NDS 서버로 DNS 질의 패킷을 보낼수 있다.
* DNS 서버에 질의 후 IP를 얻어온다
    * DNS 질의시에는 UDP 패킷이 이용된다.
* 얻어온 IP를 가지고 접속한다. 

### TCP 접속
* 3-way-handshaking 정리하기.


## Three way handshaking
### TCP FLAG
* SYN(연결 요청 플래그)	
    * TCP에서 세션을 성립할 때 가장먼저 보내는 패킷, 시퀀스 번호를 임의적으로 설정하여 세션을 연결하는 데에 사용되며 초기에 시퀀스 번호를 보내게 된다.
* ACK(응답플래그)
    * 상대방으로부터 패킷을 받았다는 걸 알려주는 패킷
    * 다른 플래그와 같이 출력되는 경우도 있습니다.
    * 받는 사람이 보낸 사람 시퀀스 번호에 TCP 계층에서 길이 또는 데이터 양을 더한 것과 같은  ACK를 보냅니다.(일반적으로 +1 하여 보냄)
    * ACK 응답을 통해 보낸 패킷에 대한 성공, 실패를 판단하여
재전송 하거나 다음 패킷을 전송한다.
* FIN(연결종료 플래그)
    * 세션 연결을 종료시킬 때 사용되며 더이상 전송할 데이터가 없음을 나타낸다.
* RST(연결 재설정 플래그)
    * 재설정(Reset)을 하는 과정이며 양방향에서 동시에 일어나는 중단 작업이다. 
    * 비 정상적인 세션 연결 끊기에 해당한다.
    * 이 패킷을 보내는 곳이 현재 접속하고 있는 곳과 즉시 연결을 끊고자 할 때 사용한다
* PSH(밀어넣기)
    * TELNET과 같은 상호작용이 중요한 프로토콜의 경우 빠른 응답이 중요한데, 이 때 받은 데이터를 즉시 목적지인 OSI 7 Layer 의 Application 계층으로 전송하도록 하는 FLAG.
    * 대화형 트랙픽에 사용되는 것으로 버퍼가 채워지기를 기다리지 않고 데이터를 전달한다.
    * 데이터는 버퍼링 없이 바로 위 계층이 아닌 7 계층의 응용프로그램으로 바로 전달한다.
* URG(긴급 데이터 플래그)
    * Urgent pointer 유효한 것인지를 나타낸다.
    * Urgent pointer란 전송하는 데이터 중에서 긴급히 전당해야 할 내용이 있을 경우에 사용한다.
    * 긴급한 데이터는 다른 데이터에 비해 우선순위가 높아야 한다. 
    *  EX) ping 명령어 실행 도중 Ctrl+c 입력

### Connection Establishment

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-11.png)

1) 클라이언트에서 서버에 SYN 패킷을 보내고 SYN_SENT 상태가 됩니다.
2) 서버는 클라이언트로부터 SYN를 받고 응답 패킷 ACK과 SYN 패킷을 패킷을 보냅니다. 상태는 LISTEN에서 SYS-SENT로 바뀝니다.
3) 클라이언트는 받은 패킷에 대한 응답으로 ACK 패킷을 보내고 상태는 ESTABLISHED가 됩니다.
4) 클라이언트로부터 ACK 패킷을 받은 서버는 상태가 ESTABLISHED로 변경됩니다.

## Four way handshaking
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-12.png)

1) 클라이언트에서 서버와의 연결 종료를 위해 서버에 FIN 패킷을 보내고 FIN_WAIT1 상태가 됩니다. (반대로 서버에서 먼저 끊을 수 도 있습니다.)
2) 서버는 클라이언트로부터 FIN을 받고 응답 패킷 ACK을 보냅니다. 상태는 CLOSE_WAIT가 됩니다
3) 서버가 통신이 끝나면, 즉 연결을 종료할 준비가 되면 클라이언트에게 FIN패킷을 보내고 LAST_WAIT 상태가 됩니다  
4) 클라이언트는 확인 패킷 ACK을 보내고 TIME_WAIT 상태가 됩니다.



## 패킷 라우팅
* NIC를 통해 외부로 나간 프레임은 라우터나 게이트웨이에 의해 라우팅 되며 스스로 목적지를 찾아간다.
    * 라우터에서 최적의 경로를 찾는 것은 RIP(routing infomation protocol) 또는 OSPF(open shortest path first), BGP(board gateway protocol)라는 프로토콜을 주기적으로 주고받으며 자신의 라우팅 테이블을 최신 정보로 갱신한다.

## 목적지 호스트 수신
* 위의 과정을 거쳐 패킷이 서버에 들어가면
* 해당 NIC 의 디바이스 드라이버가 먼저 해당 프레임의 목적지 물리주소를 확인해서 자신의 물리주소이거나 브로드캐스팅 물리주소(FF:FF:FF:FF:FF:FF)라면 해당 프레임을 받아들이고 아리면 버린다.

## TCP 접속 종료
* 4-way-핸드세이킹 과정 넣기.

## OSI 7 계층과 TCP/IP
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-13.png)

## 시스템 프로그래밍
### fork, exec 를 통한 프로세스의 실행
* fork는 자신의 복사본을 만들고
* exec는 현재 프로세스에 프로그램의 실행 이미지를 바꾸는 일을 한다. 
* fork 는 자신과 코드를 공유하는 자식 프로세스를 생성할때 사용
* exec는 새로운 프로그램을 로드해서 실행할 때 사용.

### 시그널 처리 함수
* signal 함수를 사용한 시그널 핸들러 설정
```c++
//signum : 처리하기 원하는 시그널 번호
//handler : 처리를 원하는 signum에 대한 처리 함수
#inclulde <signal.h>
sighandler_t signal(int signum, sighandler_t handler);
```

### thread 관련
* pthread 기본 API

```c++
#include <pthread.h>

//thread 생성
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void *), void * arg);

//thread 종료를 기다리는.
int pthread_join(phtread_t th, void **thread_return);

//thread를 main thread에서 분리시키기 위한.
int pthread_detach(pthread_t th);

//thread 종료.
void pthread_exit(void *retval);
```
* thread 동기화 관련 API
* 뮤텍스(mutex)

```c++
#include <pthread.h>
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *mutexattr);
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_destory(pthread_mutex_t *mutex);
```
* 세마포어(semaphore)

```c++
#include <semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_wait(sem_t *sem);
int sem_post(sem_t *sem);
int sem_destory(sem_t *sem);
```

# RAW 소켓
### 특징
* 응용계층, 전송계층, 네트워크 계층까지 모두 접근 가능
    * 네트워크 계층(IP프로토콜) 헤더와 전송 계층(TCP, UDP)헤더를 직접 제어할수있다.
    * 네트워크 계층으로 전송되는 모든 패킷을 전부 볼 수 있다. 
* 일반적으로 OS는 패킷의 목적지 IP 주소가 자신의 IP주소이면 받아들이고 아니면 버리는데
    * RAW 소켓을 사용하면 IP 주소나 포트가 다르더라도 물리 계층인 이더넷 디바이스 드라이브에서 오는 모든 패킷을 전부 받아 볼 수 있다.
* RAW 소켓으로 작성가능한 프로그램
    * ping 
        * ICMP 프로토콜 호출하여 구현가능
    * 패킷 캡쳐 프로그램
        * libpcap 가 raw 소켓을 이용해 만들어 졌다. 


# TCP 기반 서버의 호출
## 서버에서의 기본적인 함수 호출 순서
```
socket()            소켓생성
↓
bind()              소켓에 주소 할당
↓
listen()            연결 오쳥 대기 상태
↓
accept()            연결 허용
↓
read() & write()    데이터 송수신
↓
close()             연결 종료
```

## 클라이언트에서 기본적인 함수 호출 순서
```
socket()            소켓생성
↓
connect()           연결 요청
↓
read() & write()    데이터 송수신
↓
close()             연결 종료
```

## TCP 클라이언트에서 서버 함수 호출 관계
```
SERVER              CLIENT
socket()            socket()
↓                   ↓
bind()              ↓
↓                   ↓
listen()            ↓
↓       ←---        connect()
↓                   ↓
accept()            ↓
↓                   ↓
read() & write() ↔  read() & write()
↓                   ↓
close()             close()
```



## 소켓 연결 종료
### half-close
* 아래와 같은 기능을 shutdown 함수는 제공해줌

```c++
#include <sys/socket.h>
int shutdown(int s, int how);
```
* s: 종료하고자 하는 소켓의 파일 디스크립터
* how : 종료 모드
    * 0
        * SHUT_RD (입력 스트림 종료, recv buffer 만 차단한다.)
    * 1
        * SHUT_WR (출력 스트림 종료, send buffer 만 차단한다.)
    * 2
        * SHUT_RDWR (입출력 스트림 종료, 두 버퍼 모두 차단한다.)
* 그렇다면 close 와의 차이는?
    * 표준 TCP 연결은 4-way 종료에 의해 종료됩니다.
        * 참가자가 더 이상 보낼 데이터가 없으면 다른 참가자에게 FIN 패킷을 보냅니다.
        * 상대방은 FIN에 대한 ACK를 반환합니다.
        * 상대방도 데이터 전송을 마치면 다른 FIN 패킷을 보낸다.
        * 초기 참가자는 ACK를 반환하고 전송을 완료합니다.
    * 그러나 TCP 연결을 닫는 또 다른 "긴급" 방법이 있습니다.
        * 참가자가 RST 패킷을 보내고 연결을 포기합니다.
        * 상대방은 RST를 수신한 다음 연결을 포기합니다.
* shutdown(SD_SEND)의 호출은 바로 FIN을 상대에게 날림으로 4-way handshake를 유도하게 된다.
    * 이런 동작은 half-close라 하여 클라이언가 자신이 보내고 싶은 데이터를 모두 보내고 종료 할 수 있도록 해준다. (graceful shutdown)
* 차이는 (windows)
    *  shutdown(SD_SEND)은 무조건 FIN 패킷을 보내는 반면, closesocket()은 자신의 소켓버퍼 상태와 소켓옵션에 따라 RST 패킷을 보낼 수도 있고, FIN 패킷을 보낼 수도 있다.
    * shutdown()은 함수의 호출을 막는데 반해, closesocket()은 소켓 버퍼 자체를 종료 시킨다.
    * closesocket()은 socket리소스의 반환 동작까지 수행한다.


## IPC
* 굳이 이건 정리 안하기로.

## I/O multiplexing
* 멀티 프로세스
    * 여러 client에서 요청이 들어올때 
    * server 부모 프로세스에서 fork 등으로 각각의 client요청에 자식 프로세스를 생성해서 처리하기
* 멀티 플렉싱
    * 여러 client에서 요청이 들어올때 
    * server 부모 프로세스에서는 select 방식 등으로 처리하기


### select 함수
* select함수의 특징은 한곳에 여러 파일 디스크립터들을 모아놓고 동시에 관찰할 수 있다.
    * 관찰할 수 있는 항목들은 아래와 같다. 
        * 1. 수신한 데이터를 가지고 있는 소켓이 있는가? (read)
        * 2. 블로킹되지 않고 데이터의 전송이 가능한 소켓이 있는가? (write)
        * 3. 예외상황이 발생한 소켓은 무엇인가? (except)
* 파일 디스크립터 관련 기억 리마인드용(1024개의 배열)

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-14.png)
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-05-26-etc-network-15.png)


## 멀티캐스트(Multicast)
* 보통의 UDP 서버 / 클라이언트의 데이터 전송시 목적지가 하나의 호스트가 아닌
    * 둘 이상의 호스트에다 데이터를 전송하고자 하는 경우 
    * 멀티캐스트 그룹에 속한 모든 호스트들에 한번에 데이터를 전송하기 위해 사용됨.
    * 라우터 장비에서 패킷복사가 이루어지며 전달 됨.
        * 하지만, 대부분의 라우터가 멀티캐스트를 지원하지 않음.(막아놓거나.)

## 동기화(Synchronization)
### 리눅스 기반 thread를 동기화 하는데 일반적으로 2가지 방법을 사용한다
* 뮤텍스(mutex)
    * Mutual Exclusion 의 줄입말
        * thread 간에 동시 접근을 허용하지 않겠다라는 의미

```c++
#include <pthread.h>
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *mutexattr);
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_destory(pthread_mutex_t *mutex);
```

* 세마포어(semaphore)

```c++
#include <semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_wait(sem_t *sem);
int sem_post(sem_t *sem);
int sem_destory(sem_t *sem);
```
* 결론 
    * 뮤텍스(Mutex)는 이진 세마포어(Binary Semaphore)로 세마포어의 일종
    * 뮤텍스는 오직 1개의 프로세스 혹은 스레드만이 공유 자원에 접근할 수 있고, 세마포어는 지정된 변수의 값만큼 접근 가능

## 윈도우즈 기반 쓰레드 동기화
* 굳이 세부내용은 정리 안함. 굳이...
* 유저모드에서의 동기화
    * CRITICAL_SECTION 오브젝트 사용
* 커널 모드에서의 동기화
    * Mutex 커널 오브젝트 사용
    * Semaphore 커널 오브젝트 사용 
    * Event 커널 오브젝트 사용

## ETC
* WSAEventSelect 모델, Overlapped 입출력 모델, IOCP는 굳이 정리하지 않음 이제 쓸일도 없고,.



.