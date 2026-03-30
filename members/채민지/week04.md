# 1. Throughput(전송률)

## 1-1. Throughput이란?

> **송신자에서 수신자까지 실제로 데이터가 전달되는 속도**를 의미. 단위는 보통 **bits/sec**를 사용하며, throughput은 두 가지로 볼 수 있음

- **instantaneous throughput**: 어떤 순간의 전송 속도
- **average throughput**: 일정 시간 동안의 평균 전송 속도

## 1-2. 병목링크(bottleneck link)
> 종단 간 전송률은 경로 전체에서 가장 느린 링크에 의해 결정됨. 즉, 두 링크만 있는 단순한 경우 전송률은 다음처럼 생각할 수 있음
![title](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fcm9csD%2Fbtre33aK6rj%2FAAAAAAAAAAAAAAAAAAAAAE4VdMOuyD3CJaU9DLcd3Nq191YkIzA9Y86r4uaipKn2%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3DR1rvaSP%252FcQALWVdWiZRfGXkXcaE%253D)   

```
throughput = min(Rs, Rc)
```

- `Rs`: 서버 쪽 링크 전송률
- `Rc`: 클라이언트 쪽 링크 전송률

이 때 **가장 작은 값이 병목 링크의 전송률**이 되고, 전체 성능을 제한함(end-to-end throughput은 경로상 **가장 느린 링크가 결정**함)

## 1-3. 여러 링크가 있을 때
서버와 클라이언트 사이에 링크가 여러개 존재할 경우 전체 전송률은 다음과 같이 볼 수 있음

```
throughput = min(R1, R2, ..., RN)
```

![title](https://encrypted-tbn1.gstatic.com/images?q=tbn:ANd9GcSkuUd8fv3zkeImI1ygiD5-ugWtokSuddDWd1xgxdLi_yiD0GAv)   
즉, 중간에 라우터가 여러 개 있어도 결국 **가장 느린 링크**가 전체 전송률을 결정함 (책에선 이를 병목 링크가 **end-to-end throughput을 결정한다**는 식으로 설명)
## 1-4. 여러 연결이 공유할 때
하나의 공용 링크를 여러 연결이 함께 쓰면, 각 연결이 사용할 수 있는 전송률은 더 줄어듦
![title](https://phoenix.goucher.edu/~kelliher/s2011/cs325/jan31img13.png)

- PPT에선 10개의 연결이 하나의 백본 병목 링크 `R`을 공평하게 공유하는 예를 보여주며, 각 연결의 전송률을
```
min(Rc, Rs, R/10)
```
으로 설명함. 

→ 즉, 링크 자체 성능뿐 아니라 **다른 트래픽의 존재**도 throughput에 영향을 줌
## 1-5. 핵심 정리
- throughput은 **실제 데이터 전달 속도**임
- 평균 전송률과 순간 전송률을 구분해야 함
- 종단 간 전송률은 **경로 중 가장 느린 링크**에 의해 제한됨
- 여러 사용자가 같은 링크를 공유하면, **그 링크가 또 다른 병목이 됨**
# 2. Protocol Layers와 Service Models
## 2-1. 계층 구조가 필요한 이유
인터넷은 매우 복잡한 시스템임

호스트, 라우터, 다양한 링크, 수많은 프로토콜, 하드웨어와 소프트웨어가 얽혀 있기 때문에 이를 한 번에 이해하거나 설계하기 어려움 
![title](https://media.cheggcdn.com/media/85a/85a57eb3-35d6-4786-a164-4c5a29e796d7/php2Dlam9)   

∴ 네트워크를 **계층(layer)** 으로 나누어 구조화함


<details><summary>
책에서 설명을 위해 사용한 항공 여행 비유</summary>

![title](https://encrypted-tbn1.gstatic.com/images?q=tbn:ANd9GcQ2wynLLg3g3g0KZigiS5lQ_zY0WpN_Zb_HI3wwuIJLx66eNhsn)   
- 티켓 구매
- 수하물 처리
- 게이트 탑승
- 이륙/착륙
- 항공 경로 지정

▷ 이처럼 복잡한 시스템을 기능별로 나누면 각 단계의 역할을 분명히 볼 수 있음
</details>

## 2-2. 계층화의 장점
- 복잡한 시스템을 **구조적으로 이해**하기 쉬워짐
- 각 계층이 맡는 역할을 분리해 **모듈화**할 수 있음
한 계층 내부 구현을 바꿔도, 같은 서비스를 유지하면 다른 계층엔 영향을 덜 줌
> 즉, **유지보수와 업데이트가 쉬워진다**는 점이 핵심임 (ppt에선 "change in layer's service implementation: transparent to rest of system"라고 정리)
## 2-3. 계층화의 한계
책에선 계층 구조가 무조건 완벽한 건 아니라고도 말함
- 어떤 기능이 **중복 구현**될 수 있음
- 특정 기능이 사실은 다른 계층에도 필요할 수 있음
- 계층 간 구분이 항상 깔끔하진 않음
> 즉, 계층화는 매우 유용하지만, 실제 네트워크에선 **완전히 이상적인 분리**가 어려울 수 있음
# 3. 인터넷의 5계층 프로토콜 스택
![title](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbSgWKo%2Fbtre33oncgE%2FAAAAAAAAAAAAAAAAAAAAANq7f0TYPh0oztB0mUiXxPSqKlVJ6PVU66ytmekRje2M%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3DXJbEtqgQTHomo64LmUFs2ccYJGI%253D)   


인터넷 프로토콜 스택은 크게 5계층으로 구성됨 (PPT와 책 모두 같은 구조 제시)
## 3-1. Application Layer
> 네트워크 애플리케이션과 그 프로토콜이 동작하는 계층

### 대표 예시
- HTTP
- SMTP
- IMAP
- FTP
- DNS

애플리케이션 계층에서 교환되는 데이터 단위를 **message**라고 함
## 3-2. Transport Layer
> 애플리케이션 프로세스 간의 데이터 전달을 담당함

### 대표 프로토콜
- TCP: 신뢰적 전송, 흐름 제어, 혼잡 제어, 연결 지향
- UDP: 비연결형, 최소 기능, 빠르고 단순

전송 계층 데이터 단위는 **segment**임

## 3-3. Network Layer
> 한 호스트에서 다른 호스트까지 데이터를 전달하는 역할을 함
### 대표
- IP
- 라우팅 프로토콜

전송 계층 세그먼트에 네트워크 계층 정보가 붙어 **datagram**이 됨
## 3-4. Link Layer
> 인접한 네트워크 요소 사이에서 데이터를 이동시킴
### 대표
- Ethernet
- WiFi(802.11)
- PPP
- DOCSIS

링크 계층 데이터 단위는 **frame**임

경로의 각 링크마다 다른 링크 계층 프로토콜이 사용될 수 있음
## 3-5. Physical Layer
> 프레임 안의 비트를 실제 전송 매체를 통해 보내는 계층
### 예시
- 구리선
- 광섬유
- 무선 신호

즉, 물리 계층은 비트를 실제 신호로 전송하는 역할을 함
# 4. Encapsulation(캡슐화)
![title](https://velog.velcdn.com/images/dltmdrl1244/post/d7693647-738c-48f2-b81e-b7214b44543a/image.png)  
 ## 4-1. 개념
> 상위 계층의 데이터에 하위 계층이 자신의 헤더를 붙여 내려보내는 과정

![title](https://velog.velcdn.com/images/dltmdrl1244/post/be180a3e-1ba6-42b6-bf09-92268ea0f5d1/image.png)   
## 4-2. 과정
![title](https://velog.velcdn.com/images/limjung99/post/eafe8f1b-067e-40da-96ed-0ddef98eb0a6/image.png)   

송신 측에서 데이터는 다음 순서로 포장됨
```
Application: message
Transport: segment = Ht + M
Network: datagram = Hn + Ht + M
Link: frame = Hl + Hn + Ht + M
```
- `M`: application message
- `Ht`: transport header
- `Hn`: network header
- `Hl`: link header

→ 즉, 내려갈수록 헤더가 계속 붙고, 수신 측에선 반대로 각 계층이 자신의 헤더를 제거하며 원래 메시지를 복원함
## 4-3. 캡슐화가 필요한 이유
각 계층은 자신의 서비스를 제공하기 위해 필요한 제어 정보를 헤더에 담음

<details><summary>
예시
</summary>

- 전송 계층: 어느 프로세스로 보낼지
- 네트워크 계층: 어느 호스트로 보낼지
- 링크 계층: 다음 홉으로 어떻게 보낼지
</details>

→ 즉, 캡슐화는 **계층별 역할 수행을 위한 정보 부착 과정**
## 4-4. 장비별 구현 계층
- **Host**: 5계층 모두 구현
- **Router**: 보통 network/link/physical 중심
- **Link-layer switch**: 주로 link/physical

이 차이는 각 장비의 기능 차이를 반영함
# 5. Network Security (네트워크 보안)
## 5-1. 인터넷, 처음부터 안전하게 설계되지 않음
- PPT에선 인터넷이 원래 **서로 신뢰하는 사용자 집단**을 전제로 설계됐다고 설명
- 책에서도, 초기 인터넷은 상호 신뢰를 가정했기 때문에 오늘날 기준으론 보안 취약성이 크다고 설명
## 5-2. 주요 공격 유형
### (1) Malware
인터넷을 통해 악성코드가 호스트에 침투할 수 있음
- 파일 삭제
- 개인정보 탈취
- 스파이웨어 설치
- 감염된 기기를 봇넷의 일부로 활용

▷ 책에선 오늘날 많은 악성코드가 **self-replicating**해 빠르게 퍼질 수 있다고 설명
### (2) DoS / DDoS
> **Denial of Service**는 정상 사용자가 서비스를 이용하지 못하도록 만드는 공격

![title](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FxoFun%2FbtreYn2A7zB%2FAAAAAAAAAAAAAAAAAAAAAPyvYA9ZXN6jxuQ0MZ4JHzQhLei3rCbmSzCDWHTCcdg5%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3DFGdxHMCit9YbcIy%252BBT%252BfLWToXcA%253D)   
책에선 DoS 공격을 세 가지로 나눔
- vulnerability attack
- bandwidth flooding
- connection flooding

또한 여러 좀비 호스트를 동원하는 **DDoS 공격**은 단일 출처 공격보다 훨씬 탐지와 방어가 어려움 (PPT도 공격자가 여러 compromised host를 이용해 트래픽을 보내는 그림으로 설명)
### (3) Packet Sniffing
> 공격자가 전송되는 패킷을 몰래 복사해 보는 공격

![title](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FdKqCUn%2Fbtre5ckWu5H%2FAAAAAAAAAAAAAAAAAAAAAEx-lsnFZmsv_9D3s3Lg-nI_p7CQuHSQxUszfDCrnA_f%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3Dva%252BPTYhfrgmj%252BTcEIu8ef%252FbJvow%253D)   

- 무선 환경
- 공유 이더넷
- 브로드캐스트 환경

▷ 책에선 패킷 스니퍼가 **수동적(Passive)** 이라 탐지가 어렵다고 설명하며, 암호화가 중요한 방어 수단이라 말함. PPT에서도 Wireshark를 packet sniffer 예시로 제시함
### (4) IP Spoofing
> 출발지 주소를 거짓으로 꾸며 패킷을 보내는 공격

![title](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fd4QnZ7%2Fbtre1aVOovd%2FAAAAAAAAAAAAAAAAAAAAANmZfD_MAOajO0-qTFpEll-3yXTLf3rIIhOnY-PodtKp%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3DHPh32PK9UHpq6cVDsx8bbgadng4%253D) 
- 자신의 소스는 C인데, 마치 B인 것 처럼 B가 보내는 패킷으로 오해하게끔 패킷을 보내서 서버가 잘못된 처리를 유도함

- 책은 이를 **trusted party로 가장하는 문제**와 연결해서 설명하며, 해결책으로 **end-point authentication**의 필요성을 제시함
## 5-3. 방어 방법
- **authentication**: 자신이 누구인지 증명
- **confidentiality**: 암호화를 통한 기밀성
- **integrity**: 디지털 서명 등을 통한 무결성
- **access restrictions**: 접근 통제
- **firewall**: 패킷 필터링 및 공격 대응
> 즉, 보안은 특정 한 계층만의 문제가 아니라 **모든 계층에서 고려되어야 하는 요소**
# 6. 인터넷 역사
## 6-1. 1961 ~ 1972 : 패킷 교환의 시작
> 책, PPT 모두 초창기 인터넷의 뿌리를 **packet switching**에서 찾음
### 주요 내용
- Kleinrock: 대기행렬 이론으로 packet switching의 효과 설명
- Baran: 군사용 네트워크에서 packet switching 연구
- ARPAnet 구상
- 1969: 첫 ARPAnet 노드 동작
- 1972: 공개 시연, NCP, 최초의 전자메일 프로그램 등장
## 6-2. 1972~1980: 상호 연결(internetworking)
> 초기 ARPAnet은 단일 네트워크였지만, 점차 서로 다른 네트워크들을 연결하려는 필요가 생겼음
### 주요 내용
- ALOHAnet
- 패킷 위성/무선 네트워크
- Vinton Cerf, Robert Kahn의 internetworking architecture
- Ethernet 개발
- 다양한 독자 네트워크 등장

이 시기에 **network of networks**란 개념이 자리잡기 시작했음
## 6-3. 1980~1990: TCP/IP와 네트워크 확산
> 이 시기는 인터넷의 표준화와 확산의 핵심 시기임

![title](https://images.ctfassets.net/xq10wb7ogoji/3Pu3sGBRfHY5tPmBu11LCv/e55bf81d54606efd3917c5e6cd100bac/NSFmap.jpg)   

### 주요 사건
- 1983: TCP/IP 공식 배치
- SMTP 정의
- DNS 정의
- FTP 정의
- TCP congestion control
- CSnet, BITnet, NSFnet 등 확산
- 수많은 호스트가 인터넷에 연결됨
## 6-4. 1990년대 이후
- ARPAnet 종료
- 상업적 사용 허용
- Web, HTML, HTTP 등장
- Mosaic, Netscape
- 메신저, P2P, 대중화
- 이후 광대역, 무선, 스마트폰, 클라우드, SDN까지 확장
## 6-5. 역사 파트의 핵심
인터넷은 한 번에 완성된 것이 아니라,
- packet switching
- internetworking
- TCP/IP 표준화
- 웹의 대중화
- 모바일/클라우드 확산

같은 단계를 거치며 발전해 왔음

▷ 즉, 오늘날 인터넷은 **수십 년간 누적된 기술적 선택과 표준화의 결과물**임
# 7. Application Layer 입문
해당 파트에선 애플리케이션 계층 개요를 소개함
## 7-1. 애플리케이션 계층에서 다루는 것
- Web / HTTP
- E-mail / SMTP / IMAP
- DNS
- video streaming / CDN
- socket programming with UDP and TCP
## 7-2. 네트워크 앱은 어디서 동작하는가 ?
> 네트워크 앱은 **종단 시스템(end systems)** 에서 동작함

![title](https://miro.medium.com/v2/resize:fit:1200/1*QaS0t6D7v5nqYzFsLIxJhQ.png)   

- 클라이언트 프로그램
- 서버 프로그램

라우터 같은 네트워크 코어 장비엔 일반적으로 사용자 애플리케이션이 없음

∴ 네트워크 애플리케이션 개발은 **종단 시스템 중심**으로 이루어짐
## 7-3. Client-Server 패러다임
![title](data:image/jpeg;base64,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)   
### 특징
- 서버는 항상 켜져 있음
- 서버는 보통 고정 IP를 가짐
- 클라이언트는 요청을 보내고 서버와 통신
- 클라이언트끼리는 보통 직접 통신하지 않음
### 예시
- HTTP
- IMAP
- FTP
## 7-4. P2P 패러다임
### 특징
- 항상 켜져 있는 중앙 서버가 없음
- 피어들이 서로 직접 통신
- 새 피어가 들어오면 수요뿐 아니라 자원도 함께 증가
- 하지만 관리가 복잡하고 IP 변경 문제도 있음
### 예시
- BitTorrent

## 7-5. Process와 Socket
- **process**: 호스트 안에서 실행되는 프로그램
- **socket**: 프로세스가 네트워크와 통신하는 창구

PPT에서는 socket을 **문(door)** 에 비유함

앱 개발자는 socket을 통해 데이터를 보내고 받고, 실제 전송은 OS와 transport infrastructure가 담당함

## 7-6. 프로세스 식별

프로세스를 식별하려면 단순히 IP만으로는 부족함
∵ 같은 호스트에서 여러 프로세스가 실행될 수 있기 때문

∴ 식별자는 보통

- **IP address**
- **port number**

를 함께 사용

### 예시

- HTTP 서버 포트: 80
- 메일 서버 포트: 25

## 7-7. 애플리케이션 프로토콜이 정의하는 것

- 메시지의 종류
- 메시지의 문법(syntax)
- 필드의 의미(semantics)
- 언제 어떻게 송수신할지에 대한 규칙

또한 프로토콜은

- **open protocol**: RFC에 공개
- **proprietary protocol**: 기업/서비스 전용

으로 나눌 수 있음

## 7-8. 애플리케이션이 transport 계층에 요구하는 것

앱마다 필요한 서비스가 다름

- **data integrity**: 파일 전송, 웹 문서는 손실 허용 불가
- **timing**: 실시간 통화, 게임은 지연에 민감
- **throughput**: 영상 스트리밍은 일정 전송률 필요
- **security**: 암호화, 무결성 필요

## 7-9. TCP vs UDP

**TCP**

- reliable
- flow control
- congestion control
- connection-oriented

**UDP**

- unreliable
- 연결 설정 없음
- 흐름/혼잡 제어 없음
- 단순하고 빠름

애플리케이션 특성에 따라 TCP 또는 UDP를 선택함

예를 들어 **파일 다운로드나 웹**은 주로 **TCP**를 쓰고, **게임이나 일부 실시간 서비스**는 **UDP**를 사용할 수 있음

![title](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fdt5vBA%2FbtrHTEBn7fa%2FAAAAAAAAAAAAAAAAAAAAAH-37KmDB2O5bpHiPvFzEIcjC3PfRigC5ueb6VvkqEfg%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3D8UFJLoLKMnz0DKjvL6lfX60gV%252FY%253D)   

|  | **UDP** | **UDP** |
| --- | --- | --- |
| **신뢰성** | **높음** | **낮음** |
| **속도** | **낮음** | **높음** |
| **전송 방법** | **패킷이 순서대로 전달됨** | **패킷이 스트레이트로 전달됨** |
| **오류 감지 및 수정** | **있음** | **없음** |
| **혼잡도 제어** | **있음** | **없음** |
| **전송 인정** | **있음** | **오직 체크섬만** |

## 7-10. TLS

기본 TCP/UDP 소켓은 암호화를 제공하지 않음

PPT에서는 **TLS**가 TCP 위에서 동작하며,

- 암호화
- 무결성
- 종단 인증

을 제공한다고 설명함

▷ 즉, 안전한 애플리케이션 통신을 위해 TLS가 중요한 역할을 함

---

## 8. 이번 주 최종 핵심 요약

### 꼭 기억해야 할 내용

1. **Throughput은 병목 링크가 결정**
2. 여러 연결이 하나의 링크를 공유하면 각 연결 전송률은 더 감소함
3. 인터넷은 복잡한 시스템이라 **계층 구조**로 이해함
4. 인터넷 프로토콜 스택은 **Application / Transport / Network / Link / Physical**의 5계층임
5. 캡슐화는 상위 계층 데이터에 하위 계층 헤더를 붙이는 과정임
6. 인터넷은 원래 보안을 충분히 고려하지 않고 설계되어서 다양한 공격에 취약함
7. 대표적인 공격은 **malware, DoS/DDoS, sniffing, spoofing**
8. 애플리케이션 계층에서는 HTTP, DNS, 이메일, 스트리밍, 소켓 프로그래밍 등을 다룸
9. 애플리케이션마다 필요한 transport 서비스가 다르므로 TCP와 UDP의 선택 기준이 달라짐