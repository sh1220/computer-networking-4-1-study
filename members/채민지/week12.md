# 1. Network Layer와 IP

## 1-1. Network Layer: Internet

- **Network layer** : host와 router에서 packet을 목적지까지 전달하기 위한 계층
- Internet의 network layer는 크게 **IP protocol**, **ICMP protocol**, **routing / path-selection 기능**으로 구성됨

주요 구성은 다음과 같음

- **IP protocol** : datagram format, addressing, packet handling conventions 담당
- **ICMP protocol** : error reporting, router signaling 담당
- **Forwarding table** : router가 packet을 어느 output link로 보낼지 결정할 때 사용하는 table
- **Path-selection algorithms** : packet이 목적지까지 갈 경로를 결정하는 알고리즘
- **Routing protocols** : OSPF, BGP 같은 경로 결정 프로토콜
- **SDN controller** : 중앙 controller가 forwarding rule을 계산하는 구조에서 사용

⇒ Internet network layer는 **IP datagram을 만들고, 주소를 붙이고, router들이 forwarding table을 이용해 목적지 방향으로 보내는 구조**

---

# 2. IP Datagram Format

## 2-1. IP Datagram

- **IP datagram** : network layer에서 사용하는 packet 단위
- 보통 transport layer의 TCP segment나 UDP segment를 payload로 담음
- IPv4 datagram은 **header + payload data**로 구성됨

IP datagram header의 주요 필드는 다음과 같음

- **Version** : IP protocol version 번호
- **Header length** : IP header 길이
- **Type of service** : 서비스 유형 표시
- **Datagram length** : IP datagram 전체 길이
- **16-bit identifier** : fragmentation / reassembly에 사용되는 식별자
- **Flags** : fragmentation 관련 제어 정보
- **Fragment offset** : 조각난 datagram의 위치 정보
- **TTL** : packet이 지나갈 수 있는 최대 hop 수
- **Upper layer** : payload가 TCP인지 UDP인지 나타내는 필드
- **Header checksum** : IP header 오류 검출
- **Source IP address** : 송신자 IP 주소
- **Destination IP address** : 수신자 IP 주소
- **Options** : 선택적 기능
- **Payload data** : 실제로 전달되는 데이터

⇒ IP datagram header는 **packet이 목적지까지 가는 데 필요한 주소, 길이, 수명, 상위 계층 정보 등을 담는 부분**

---

## 2-2. TTL

- **TTL** : Time To Live
- packet이 network 안에서 무한히 떠돌지 않도록 제한하는 필드
- router를 하나 지날 때마다 TTL 값이 1씩 감소함
- TTL이 0이 되면 packet은 폐기됨

⇒ TTL은 **routing loop 등으로 packet이 무한히 돌아다니는 것을 막기 위한 장치**

---

## 2-3. Upper Layer Field

- **Upper layer field** : IP payload를 어느 transport layer protocol에게 넘길지 알려주는 필드
- 예시로 TCP, UDP 등이 있음
- 수신 host는 이 값을 보고 IP payload를 TCP 또는 UDP로 넘김

⇒ Upper layer field는 **IP와 transport layer를 연결해주는 구분자**

---

## 2-4. Header Checksum

- **Header checksum** : IP header에 오류가 있는지 확인하기 위한 필드
- router는 header를 검사하고, 오류가 있으면 datagram을 폐기할 수 있음
- IPv4에는 존재하지만 IPv6에서는 제거됨

⇒ Header checksum은 **IPv4 header 오류 검출용 필드**

---

## 2-5. IP Header Overhead

- IPv4 header는 options가 없으면 보통 **20 bytes**
- TCP header도 보통 **20 bytes**
- 따라서 TCP/IP를 사용할 경우 기본적으로 **40 bytes + application layer overhead** 발생

⇒ TCP/IP 통신에서는 실제 데이터 외에도 header로 인한 overhead 존재

---

# 3. IP Addressing

## 3-1. IP Address

- **IP address** : host나 router interface에 부여되는 32-bit identifier
- IPv4 주소는 32bit로 구성됨
- 사람이 보기 쉽게 **dotted-decimal notation**으로 표현함

예시

- `223.1.1.1`
- binary 표현 : `11011111 00000001 00000001 00000001`

각 8bit를 decimal로 바꾸면 다음과 같음

- `223.1.1.1`

⇒ IP address는 **host 자체가 아니라 interface에 붙는 주소**

---

## 3-2. Interface

- **Interface** : host/router와 physical link 사이의 연결 지점
- router는 보통 여러 interface를 가짐
- host는 보통 하나 또는 두 개의 interface를 가짐

예시

- 노트북의 wired Ethernet interface
- 노트북의 wireless WiFi interface
- router의 여러 network 연결 interface

⇒ IP address는 host 전체에 하나 붙는 것이 아니라, **각 interface마다 부여되는 주소**

---

## 3-3. Interface 연결 방식

- host나 router의 interface들은 실제로 여러 방식으로 연결될 수 있음
- 유선 Ethernet interface는 Ethernet switch를 통해 연결될 수 있음
- wireless WiFi interface는 WiFi base station을 통해 연결될 수 있음

⇒ IP 관점에서는 우선 interface들이 어떤 subnet에 속하는지가 중요하고, 실제 link layer 연결 방식은 이후 계층에서 다룸

---

# 4. Subnet

## 4-1. Subnet

- **Subnet** : intervening router를 거치지 않고 서로 물리적으로 도달 가능한 device interface들의 집합
- 같은 subnet에 있는 device들은 IP address의 앞부분, 즉 subnet part를 공유함
- IP address는 크게 **subnet part + host part**로 나눌 수 있음

구성은 다음과 같음

- **Subnet part** : 같은 subnet에 속한 device들이 공유하는 상위 bit
- **Host part** : subnet 안에서 각 interface를 구분하는 하위 bit

⇒ Subnet은 **router를 거치지 않고 직접 도달 가능한 interface들의 network 단위**

---

## 4-2. Subnet Mask

- **Subnet mask** : IP address에서 어디까지가 subnet part인지 나타내는 정보
- `/24`는 상위 24bit가 subnet part라는 의미
- 나머지 8bit는 host part

예시

- `223.1.1.0/24`
- subnet part : `223.1.1`
- host part : 마지막 8bit

⇒ `/24`는 **앞의 24bit가 같은 device들이 같은 subnet에 속한다는 의미**

---

## 4-3. Subnet 구분 방법

Subnet을 찾는 방법은 다음과 같음

1. 각 interface를 host나 router에서 분리한다고 생각함
2. router를 거치지 않고 연결된 interface끼리 묶음
3. 이렇게 생긴 각각의 isolated network를 subnet으로 봄

예시 subnet

- `223.1.1.0/24`
- `223.1.2.0/24`
- `223.1.3.0/24`

⇒ Subnet을 찾을 때 핵심은 **router 없이 직접 도달 가능한 interface 묶음인지 확인하는 것**

---

# 5. CIDR

## 5-1. CIDR

- **CIDR** : Classless InterDomain Routing
- 기존 class 기반 주소 구분 대신, subnet part 길이를 자유롭게 정하는 방식
- 주소 형식은 `a.b.c.d/x`

의미는 다음과 같음

- `a.b.c.d` : IP address
- `/x` : 앞에서부터 x bit가 subnet part

예시

- `200.23.16.0/23`
- 상위 23bit : subnet part
- 나머지 bit : host part

⇒ CIDR은 **IP address의 network 부분 길이를 유연하게 정할 수 있는 주소 표현 방식**

---

## 5-2. CIDR의 장점

- class A/B/C처럼 고정된 크기로 주소를 나누지 않아도 됨
- 필요한 크기에 맞게 주소 block을 나눌 수 있음
- 주소 공간을 더 효율적으로 사용할 수 있음
- route aggregation과도 연결됨

⇒ CIDR은 **IPv4 주소 공간을 효율적으로 쓰고 routing 정보를 줄이기 위한 방식**

---

# 6. IP Address 할당

## 6-1. Host는 IP Address를 어떻게 얻는가

Host가 IP address를 얻는 방법은 크게 두 가지임

1. 관리자가 config file에 직접 설정
2. DHCP를 통해 동적으로 주소 획득

### 직접 설정

- sysadmin이 host의 IP address를 직접 설정하는 방식
- UNIX 계열에서는 config file에 설정 가능

### DHCP

- **DHCP** : Dynamic Host Configuration Protocol
- host가 network에 접속할 때 server로부터 IP address를 자동으로 받는 방식
- plug-and-play 방식

⇒ 일반 사용자는 보통 DHCP를 통해 자동으로 IP address를 받음

---

## 6-2. Network는 IP Address Block을 어떻게 얻는가

- Network는 ISP로부터 address block을 할당받음
- ISP는 자신이 가진 address space를 여러 organization에 나누어 줄 수 있음

예시

- ISP block : `200.23.16.0/20`
- Organization 0 : `200.23.16.0/23`
- Organization 1 : `200.23.18.0/23`
- Organization 2 : `200.23.20.0/23`

⇒ Organization은 보통 **provider ISP의 address space 일부를 할당받아 사용**

---

## 6-3. ICANN

- **ICANN** : Internet Corporation for Assigned Names and Numbers
- IP address block 할당과 DNS root zone 관리를 담당하는 기관
- 5개의 regional registry를 통해 IP address를 배분함
- TLD 관리 위임도 담당함

⇒ ICANN은 **전 세계 Internet 주소와 DNS root 관련 관리를 담당하는 기관**

---

# 7. DHCP

## 7-1. DHCP

- **DHCP** : Dynamic Host Configuration Protocol
- host가 network에 join할 때 network server로부터 IP address를 동적으로 얻는 프로토콜
- DHCP server는 보통 router 안에 함께 위치할 수 있음
- DHCP는 mobile user처럼 network에 들어오고 나가는 host를 지원하기 좋음

DHCP의 특징은 다음과 같음

- IP address를 자동 할당
- address lease 갱신 가능
- 사용하지 않는 address 재사용 가능
- host가 network에 연결된 동안만 address 사용 가능
- plug-and-play 지원

⇒ DHCP는 **host가 network에 접속할 때 필요한 설정 정보를 자동으로 받게 해주는 프로토콜**

---

## 7-2. DHCP 4단계 동작

DHCP 동작은 크게 네 단계로 정리 가능

1. **DHCP discover**
2. **DHCP offer**
3. **DHCP request**
4. **DHCP ACK**

### 1단계: DHCP Discover

- client가 DHCP server를 찾기 위해 broadcast 전송
- client는 아직 자기 IP address를 모르므로 source IP를 `0.0.0.0`으로 사용
- destination IP는 broadcast 주소 `255.255.255.255`

의미는 다음과 같음

- “이 network에 DHCP server 있음?”

### 2단계: DHCP Offer

- DHCP server가 client에게 사용 가능한 IP address를 제안
- broadcast로 응답 가능
- IP address와 lease time 등을 포함함

의미는 다음과 같음

- “내가 DHCP server임. 이 IP address 사용 가능함.”

### 3단계: DHCP Request

- client가 제안받은 IP address를 사용하겠다고 요청
- 여러 server가 offer할 수 있으므로, client가 선택한 server와 address를 알림

의미는 다음과 같음

- “이 IP address를 사용하고 싶음.”

### 4단계: DHCP ACK

- DHCP server가 최종적으로 IP address 사용을 승인
- client는 이때부터 해당 IP address 사용 가능

의미는 다음과 같음

- “그 IP address 사용해도 됨.”

⇒ DHCP는 **Discover → Offer → Request → ACK 순서로 IP address를 자동 할당하는 구조**

---

## 7-3. DHCP Message 예시

DHCP client가 처음 들어온 경우 메시지 흐름은 다음과 같음

### DHCP Discover

- source : `0.0.0.0`, port 68
- destination : `255.255.255.255`, port 67
- yiaddr : `0.0.0.0`

### DHCP Offer

- source : DHCP server IP, port 67
- destination : `255.255.255.255`, port 68
- yiaddr : client에게 제안하는 IP address
- lifetime : lease time

### DHCP Request

- source : `0.0.0.0`, port 68
- destination : `255.255.255.255`, port 67
- yiaddr : 사용하려는 IP address

### DHCP ACK

- source : DHCP server IP, port 67
- destination : `255.255.255.255`, port 68
- yiaddr : client에게 할당된 IP address

⇒ DHCP 초기 과정에서는 client가 아직 IP를 모르기 때문에 broadcast가 많이 사용됨

---

## 7-4. DHCP가 제공하는 정보

DHCP는 IP address만 주는 것이 아님

함께 제공 가능한 정보는 다음과 같음

- client의 IP address
- first-hop router address
- DNS server name
- DNS server IP address
- network mask

⇒ DHCP는 **IP address뿐 아니라 network 사용에 필요한 기본 설정 정보를 함께 제공**

---

## 7-5. DHCP Encapsulation

- DHCP message는 여러 계층을 거쳐 encapsulation됨
- DHCP request message는 UDP 안에 들어감
- UDP는 IP 안에 들어감
- IP datagram은 Ethernet frame 안에 들어감
- Ethernet frame은 LAN에서 broadcast됨

구조는 다음과 같음

`DHCP → UDP → IP → Ethernet → Physical`

수신 측에서는 반대로 demultiplexing됨

`Ethernet → IP → UDP → DHCP`

⇒ DHCP는 application layer protocol이지만 실제 전송될 때는 UDP/IP/Ethernet 계층을 거쳐 전달됨

---

# 8. Hierarchical Addressing

## 8-1. Hierarchical Addressing

- **Hierarchical addressing** : IP address를 계층적으로 배분하는 방식
- ISP가 큰 address block을 받고, 이를 organization에 작은 block으로 나누어 줌
- 이 구조 덕분에 routing information을 효율적으로 광고할 수 있음

예시

- ISP가 `200.23.16.0/20` block 보유
- 여러 organization에 `/23` 단위로 나누어 할당
- Internet에는 ISP가 큰 block 하나로 광고 가능

⇒ Hierarchical addressing은 **routing table 크기를 줄이기 위한 주소 배분 구조**

---

## 8-2. Route Aggregation

- **Route aggregation** : 여러 작은 network route를 하나의 큰 prefix로 묶어 광고하는 방식
- ISP가 자신에게 속한 organization 주소들을 하나의 prefix로 묶어 알릴 수 있음

예시

- Organization 0 : `200.23.16.0/23`
- Organization 1 : `200.23.18.0/23`
- Organization 2 : `200.23.20.0/23`
- ISP 광고 : `200.23.16.0/20`

⇒ Route aggregation은 **여러 경로 정보를 하나로 압축해 routing table을 줄이는 방식**

---

## 8-3. More Specific Route

- **More specific route** : 더 긴 prefix를 가진 더 구체적인 route
- 어떤 organization이 ISP를 옮기는 경우, 기존 큰 prefix와 별도로 더 구체적인 route를 광고할 수 있음

예시 상황

- Organization 1이 기존 ISP에서 다른 ISP로 이동
- 새 ISP가 `200.23.18.0/23`이라는 more specific route 광고
- Internet router는 longest prefix matching에 따라 더 구체적인 `/23` route를 선택

⇒ More specific route는 **큰 address block 안의 특정 network만 따로 다른 경로로 보내기 위한 방법**

---

# 9. NAT

## 9-1. NAT

- **NAT** : Network Address Translation
- local network 내부의 여러 device가 외부 Internet에서는 하나의 public IPv4 address를 공유하는 방식
- home network, institutional network, 4G/5G cellular network 등에서 많이 사용됨

예시

- 내부 network : `10.0.0.0/24`
- NAT public IP : `138.76.29.7`
- 외부 Internet 입장 : local network 전체가 하나의 IP address처럼 보임

⇒ NAT는 **여러 내부 device가 하나의 public IPv4 address를 공유하게 해주는 기술**

---

## 9-2. Private IP Address

- NAT 내부 local network에서는 private IP address 사용 가능
- private IP address는 local network 안에서만 사용됨
- 외부 Internet에서는 직접 routing되지 않음

대표 private address space는 다음과 같음

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

⇒ Private IP는 **local network 내부에서만 의미 있는 주소**

---

## 9-3. NAT의 장점

NAT의 장점은 다음과 같음

- ISP로부터 하나의 public IP address만 받아도 여러 device 사용 가능
- local network 내부 host 주소를 바꿔도 외부에 알릴 필요 없음
- ISP를 바꿔도 내부 device 주소를 바꿀 필요 없음
- 내부 device가 외부에서 직접 보이지 않아 보안상 이점 가능
- IPv4 주소 부족 문제 완화

⇒ NAT는 **IPv4 주소 절약과 local network 관리 편의성을 제공**

---

## 9-4. NAT 동작 방식

NAT router는 outgoing datagram과 incoming datagram의 주소/포트를 바꿔줌

### Outgoing Datagram

- 내부 host가 외부 server로 datagram 전송
- NAT router가 source IP와 source port를 변경함

예시

- 원래 source : `10.0.0.1, 3345`
- 변경 source : `138.76.29.7, 5001`

NAT router는 이 mapping을 NAT translation table에 저장함

### Incoming Datagram

- 외부 server가 NAT public IP와 port로 reply 전송
- NAT router가 destination IP와 destination port를 table을 보고 내부 host 정보로 변경함

예시

- 원래 destination : `138.76.29.7, 5001`
- 변경 destination : `10.0.0.1, 3345`

⇒ NAT router는 **주소와 port를 변환하고 translation table로 내부 host를 구분**

---

## 9-5. NAT Translation Table

- **NAT translation table** : 내부 address/port와 외부 NAT address/port의 mapping을 저장하는 table

예시

| WAN side addr | LAN side addr |
| --- | --- |
| `138.76.29.7, 5001` | `10.0.0.1, 3345` |

이 table이 필요한 이유는 다음과 같음

- 외부에서는 모든 packet이 NAT public IP로 들어옴
- NAT router는 port 번호를 보고 어떤 내부 host로 보낼지 결정함
- 내부 host가 여러 개여도 port mapping으로 구분 가능

⇒ NAT translation table은 **public IP 하나로 여러 내부 연결을 구분하기 위한 핵심 자료구조**

---

## 9-6. NAT 논란

NAT는 널리 사용되지만 논란도 있음

주요 논점은 다음과 같음

- router는 원래 layer 3까지만 처리해야 한다는 주장
- NAT는 port number까지 수정하므로 transport layer 정보에 개입함
- IPv4 주소 부족 문제는 IPv6로 해결해야 한다는 주장
- end-to-end argument 위반 가능
- NAT 뒤에 있는 server에 외부 client가 접속하려면 NAT traversal 문제 발생

⇒ NAT는 실용적으로는 많이 쓰이지만, **계층 구조와 end-to-end 원칙 측면에서는 논란이 있는 기술**

---

# 10. IPv6

## 10-1. IPv6 Motivation

- **IPv6** : IPv4 주소 부족 문제를 해결하기 위해 등장한 새로운 IP protocol
- IPv4는 32-bit address space를 사용함
- IPv6는 128-bit address space를 사용함

IPv6의 주요 동기는 다음과 같음

- 32-bit IPv4 address space 고갈
- router processing / forwarding 속도 개선
- flow별 다른 network-layer 처리 가능성 제공

⇒ IPv6의 가장 큰 목적은 **주소 공간 확대와 더 단순한 header 구조를 통한 처리 효율 개선**

---

## 10-2. IPv6 Datagram Format

IPv6 datagram의 주요 필드는 다음과 같음

- **Version** : IP version 번호
- **Priority / Traffic class** : datagram priority 표시
- **Flow label** : 같은 flow에 속한 datagram 식별
- **Payload length** : payload 길이
- **Next header** : 다음 header 또는 상위 protocol 표시
- **Hop limit** : IPv4의 TTL과 유사한 역할
- **Source address** : 128-bit 송신자 주소
- **Destination address** : 128-bit 수신자 주소
- **Payload data** : 실제 데이터

⇒ IPv6 header는 **128-bit 주소와 flow 처리를 위한 필드를 포함한 고정 길이 중심 구조**

---

## 10-3. IPv6에서 빠진 것

IPv6는 IPv4와 비교했을 때 일부 기능을 제거함

빠진 항목은 다음과 같음

- **Checksum 없음**
- **Fragmentation / reassembly 없음**
- **Options 없음**

### Checksum 제거

- router 처리 속도 향상을 위한 목적
- link layer와 transport layer에서도 오류 검사가 가능하기 때문

### Fragmentation / Reassembly 제거

- router가 중간에서 fragmentation하지 않음
- 필요하면 source와 destination 쪽에서 처리

### Options 제거

- 기본 header에서 options 제거
- 필요 기능은 next header 방식으로 처리

⇒ IPv6는 **router가 packet을 더 빠르게 처리할 수 있도록 header를 단순화한 구조**

---

## 10-4. IPv6 Adoption

- IPv6는 오래전부터 등장했지만 완전한 전환에는 시간이 오래 걸림
- PPT에서는 2023년 기준 Google 서비스 접근 client 중 약 40%가 IPv6 사용
- NIST 기준 미국 정부 domain 중 약 1/3이 IPv6 capable이라고 제시됨
- IPv6 배포와 사용에는 25년 이상이 걸리고 있음

⇒ IPv6는 필요성은 크지만, 기존 IPv4 infrastructure 때문에 전환이 매우 느린 기술**

---

# 11. IPv4에서 IPv6로의 전환

## 11-1. Transition Problem

- 모든 router를 한 번에 IPv6로 업그레이드할 수 없음
- Internet에는 IPv4 router와 IPv6 router가 섞여 존재함
- 따라서 IPv4-only network를 지나 IPv6 datagram을 전달하는 방법 필요

문제는 다음과 같음

- IPv6 host끼리 통신하고 싶음
- 중간 network 일부는 IPv4만 지원함
- IPv6 datagram을 그대로 전달할 수 없음

⇒ IPv4와 IPv6가 섞인 환경에서는 **transition mechanism** 필요

---

## 11-2. Tunneling

- **Tunneling** : IPv6 datagram을 IPv4 datagram의 payload로 넣어 전달하는 방식
- IPv4 network를 통과할 때 IPv6 packet을 IPv4 packet 안에 감싸서 보냄
- “packet within a packet” 구조

동작 방식은 다음과 같음

1. IPv6 router가 IPv6 datagram 생성
2. IPv4-only network를 만나면 IPv6 datagram을 IPv4 datagram 안에 encapsulation
3. IPv4 network 내부에서는 IPv4 datagram처럼 전달
4. IPv4 tunnel 끝에서 IPv6 datagram을 꺼냄
5. 이후 IPv6 network에서 다시 IPv6 datagram으로 전달

⇒ Tunneling은 **IPv6 packet을 IPv4 network 안에서 운반하기 위해 IPv4 packet으로 감싸는 방식**

---

## 11-3. Tunneling의 주소 구조

Tunneling에서는 바깥쪽 IPv4 header와 안쪽 IPv6 header가 따로 존재함

- 바깥쪽 IPv4 source/destination : tunnel 양 끝 IPv4/IPv6 router 주소
- 안쪽 IPv6 source/destination : 실제 통신하는 IPv6 host 주소

예시

- 실제 IPv6 flow : A → F
- IPv4 tunnel 구간 : B → E
- IPv4 header source : B
- IPv4 header destination : E
- IPv6 header source : A
- IPv6 header destination : F

⇒ Tunneling에서는 **겉 packet의 주소와 안쪽 packet의 주소가 다를 수 있음**

---

# 12. Generalized Forwarding

## 12-1. Match Plus Action

- **Generalized forwarding** : packet header의 여러 field를 match한 뒤, 정해진 action을 수행하는 forwarding 방식
- 기본 추상화는 **match + action**
- 기존 destination-based forwarding보다 더 일반적인 구조

### Match

- arriving packet header field 값이 flow table의 조건과 맞는지 확인하는 과정
- link layer, network layer, transport layer field 모두 match 대상 가능

### Action

- match된 packet에 대해 수행할 동작

가능한 action은 다음과 같음

- forward
- drop
- modify
- log
- copy
- send to controller

⇒ Generalized forwarding은 **header field 조건에 따라 다양한 packet 처리 동작을 수행하는 방식**

---

## 12-2. Destination-based Forwarding과의 차이

- **Destination-based forwarding** : destination IP address만 기준으로 forwarding
- **Generalized forwarding** : 여러 header field를 기준으로 forwarding, drop, modify 등 다양한 action 수행

차이는 다음과 같음

- Destination-based : 목적지 IP 중심
- Generalized : header field 전반 사용 가능
- Destination-based : 주로 forward
- Generalized : forward 외에도 drop, modify, controller 전달 가능

⇒ Generalized forwarding은 **destination IP만 보는 전통적 forwarding을 확장한 방식**

---

# 13. Flow Table

## 13-1. Flow

- **Flow** : header field 값들로 정의되는 packet들의 묶음
- link layer, network layer, transport layer field를 기준으로 flow 정의 가능

예시 기준

- source IP
- destination IP
- TCP/UDP source port
- TCP/UDP destination port
- MAC address
- ingress port
- VLAN ID

⇒ Flow는 **특정 header field 조건을 공유하는 packet 집합**

---

## 13-2. Flow Table

- **Flow table** : router나 switch가 match + action 규칙을 저장하는 table
- generalized forwarding에서 forwarding table은 flow table이라고도 부름
- flow table entry는 packet 처리 규칙을 정의함

Flow table entry의 주요 구성은 다음과 같음

- **Match** : 어떤 packet이 이 rule에 해당하는지 정의
- **Action** : 해당 packet에 수행할 동작
- **Priority** : 여러 rule이 동시에 match될 때 어떤 rule을 우선할지 결정
- **Counters** : match된 packet 수와 byte 수 기록

⇒ Flow table은 **packet header 조건과 처리 동작을 연결한 규칙표**

---

## 13-3. Wildcard

- **Wildcard** : 특정 field 값을 신경 쓰지 않는다는 표시
- 보통 로 표현됨
- flow table에서 넓은 범위의 packet을 match할 때 사용

예시

- `src = 10.1.2.3, dest = *.*.*.*`
- source가 `10.1.2.3`이면 destination은 무엇이든 match

⇒ Wildcard는 **일부 field만 조건으로 사용하고 나머지는 무시하는 match 방식**

---

# 14. OpenFlow

## 14-1. OpenFlow

- **OpenFlow** : generalized forwarding과 match + action 구조를 구현하는 대표적인 방식
- flow table entry를 기반으로 packet 처리
- SDN에서 controller가 switch/router의 flow table을 설정하는 데 사용됨

OpenFlow flow table entry의 구성은 다음과 같음

- **Match**
- **Action**
- **Stats**

⇒ OpenFlow는 **flow table 기반으로 network device의 packet 처리 규칙을 제어하는 방식**

---

## 14-2. OpenFlow Match Fields

OpenFlow에서는 여러 계층의 header field를 match 가능

### Link Layer

- Ingress port
- Source MAC
- Destination MAC
- Ethernet type
- VLAN ID
- VLAN priority

### Network Layer

- IP source
- IP destination
- IP protocol
- IP ToS

### Transport Layer

- TCP/UDP source port
- TCP/UDP destination port

⇒ OpenFlow는 **link, network, transport layer의 여러 field를 조건으로 packet 처리 가능**

---

## 14-3. OpenFlow Actions

OpenFlow의 대표 action은 다음과 같음

1. **Forward packet to port(s)**
2. **Drop packet**
3. **Modify fields in header(s)**
4. **Encapsulate and forward to controller**

⇒ OpenFlow action은 **packet을 보내거나, 버리거나, 수정하거나, controller로 전달하는 동작**

---

## 14-4. OpenFlow Stats

- **Stats** : flow table entry에 match된 packet과 byte 수를 기록하는 정보
- traffic monitoring과 관리에 사용 가능

Stats의 예시는 다음과 같음

- packet counter
- byte counter

⇒ Stats는 **각 flow rule이 얼마나 사용되었는지 확인하기 위한 정보**

# 15. OpenFlow 예시

## 15-1. Destination-based Forwarding 예시

- 목적지 IP address가 `51.6.0.8`인 IP datagram은 output port 6으로 전달
- 이 경우 match 조건은 destination IP address
- action은 `forward(port6)`

⇒ OpenFlow로도 전통적인 destination-based forwarding 구현 가능

---

## 15-2. Firewall 예시 1: TCP Port 차단

- destination TCP port가 22번인 packet을 drop
- TCP port 22는 ssh port로 사용됨
- action은 `drop`

⇒ OpenFlow flow table을 이용해 **특정 port 기반 차단 가능**

---

## 15-3. Firewall 예시 2: 특정 Host 차단

- source IP가 `128.119.1.1`인 datagram을 drop
- 특정 host에서 보낸 packet을 차단하는 규칙

⇒ OpenFlow는 **IP 주소 기반 firewall 역할도 수행 가능**

---

## 15-4. Layer 2 Forwarding 예시

- destination MAC address가 `22:A7:23:11:E1:02`인 frame을 output port 3으로 전달
- IP가 아니라 MAC address를 기준으로 forwarding

⇒ OpenFlow는 **layer 2 switch 동작도 match + action으로 표현 가능**

---

# 16. OpenFlow Abstraction

## 16-1. Match + Action으로 여러 장비 통합

OpenFlow의 match + action 추상화는 여러 종류의 network device 동작을 하나의 방식으로 설명할 수 있음

### Router

- Match : longest destination IP prefix
- Action : 특정 link로 forward

### Switch

- Match : destination MAC address
- Action : forward 또는 flood

### Firewall

- Match : IP address와 TCP/UDP port number
- Action : permit 또는 deny

### NAT

- Match : IP address와 port
- Action : address와 port rewrite

⇒ OpenFlow abstraction은 **router, switch, firewall, NAT의 동작을 match + action으로 통합해서 표현하는 방식**

---

# 17. Network-wide Behavior

## 17-1. Orchestrated Flow Tables

- 여러 switch의 flow table을 controller가 함께 설정하면 network-wide behavior를 만들 수 있음
- 하나의 device가 아니라 network 전체의 forwarding 경로를 의도적으로 구성 가능

예시

- h5, h6에서 h3, h4로 가는 datagram을 s1을 거쳐 s2로 보내도록 설정
- 각 switch의 flow table에 서로 맞물리는 rule을 설치
- 결과적으로 특정 source/destination traffic이 원하는 경로를 따라 이동

⇒ Flow table을 여러 장비에 조합하면 **network 전체 동작을 프로그램하듯 설정 가능**

---

## 17-2. Network Programmability

- **Network programmability** : network 동작을 rule 기반으로 프로그래밍하듯 제어하는 개념
- generalized forwarding은 간단한 형태의 network programmability로 볼 수 있음
- packet마다 header field를 보고 programmable processing 가능

PPT에서 언급된 관련 흐름은 다음과 같음

- historical roots : active networking
- modern direction : P4 같은 더 일반화된 programming 방식

⇒ Generalized forwarding은 **네트워크 장비의 packet 처리 방식을 유연하게 프로그래밍하는 기반**

---

# 18. 핵심 비교 정리

## 18-1. IPv4 vs IPv6

### IPv4

- 32-bit address
- header checksum 존재
- router fragmentation 가능
- options field 존재
- 주소 부족 문제 존재

### IPv6

- 128-bit address
- header checksum 없음
- router fragmentation 없음
- options를 기본 header에서 제거
- flow label 존재
- 고정 길이 40-byte header

⇒ IPv6는 **주소 공간을 크게 늘리고 router 처리 효율을 높인 IP 버전**

---

## 18-2. Subnet vs CIDR

### Subnet

- router를 거치지 않고 직접 도달 가능한 interface들의 묶음
- IP address의 subnet part를 공유함

### CIDR

- `a.b.c.d/x` 형식으로 network prefix 길이를 표현하는 방식
- subnet part 길이를 유연하게 설정 가능

⇒ Subnet은 **network 단위**, CIDR은 **주소 prefix를 표현하는 방식**

---

## 18-3. DHCP vs NAT

### DHCP

- host에게 IP address와 network 설정 정보를 자동 할당
- 내부 host가 network에 join할 때 사용
- Discover → Offer → Request → ACK

### NAT

- 내부 여러 host가 하나의 public IP address를 공유
- address와 port를 변환
- NAT translation table 사용

⇒ DHCP는 **주소를 받는 기술**, NAT는 **주소를 변환해 공유하는 기술**

---

## 18-4. Destination-based Forwarding vs Generalized Forwarding

### Destination-based Forwarding

- destination IP address만 기준으로 forwarding
- 전통적 router forwarding 방식

### Generalized Forwarding

- 여러 header field를 기준으로 match
- forward, drop, modify, send to controller 등 다양한 action 가능

⇒ Generalized forwarding은 **기존 forwarding보다 더 유연한 match + action 기반 방식**

---

## 18-5. Route Aggregation vs More Specific Route

### Route Aggregation

- 여러 작은 network prefix를 하나의 큰 prefix로 묶어 광고
- routing table 크기 감소

### More Specific Route

- 더 긴 prefix를 가진 구체적인 route
- 같은 주소 범위 안에서도 특정 network만 다른 경로로 보낼 수 있음

⇒ Route aggregation은 **경로 정보 압축**, more specific route는 **특정 경로 예외 처리**

---

# 19. 한 줄 핵심 정리

- **IP datagram** : network layer에서 전달되는 기본 packet 단위
- **IP address** : host/router interface에 부여되는 32-bit identifier
- **Subnet** : router 없이 서로 직접 도달 가능한 interface들의 집합
- **CIDR** : `a.b.c.d/x` 형식의 유연한 prefix 기반 주소 표현
- **DHCP** : host가 network 설정 정보를 자동으로 얻는 프로토콜
- **Hierarchical addressing** : 주소를 계층적으로 나눠 routing 정보를 줄이는 방식
- **NAT** : 여러 내부 host가 하나의 public IP address를 공유하는 주소 변환 기술
- **IPv6** : 128-bit 주소와 단순화된 header를 가진 IP protocol
- **Tunneling** : IPv6 datagram을 IPv4 datagram 안에 넣어 전달하는 전환 방식
- **Generalized forwarding** : 여러 header field를 match해 다양한 action을 수행하는 forwarding 방식
- **OpenFlow** : match + action 기반 flow table로 packet 처리 규칙을 제어하는 방식