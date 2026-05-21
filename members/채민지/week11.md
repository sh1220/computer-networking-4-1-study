# 1. Router Architecture

## 1-1. Router Architecture Overview

- **Router** : 입력 링크로 들어온 packet을 적절한 출력 링크로 전달하는 network layer 장비
- Router 내부 구조 : **input port + switch fabric + output port + routing processor**
- Router의 핵심 역할 : packet forwarding과 routing/management 기능 수행

구성요소는 다음과 같음

- **Input port** : packet을 입력 링크에서 받아 처리하는 부분
- **Switch fabric** : input port에서 output port로 packet을 이동시키는 내부 연결 구조
- **Output port** : packet을 출력 링크로 내보내기 위해 저장, 스케줄링, 전송하는 부분
- **Routing processor** : routing protocol 실행, forwarding table 계산, router 관리 기능 수행

⇒ Router는 단순히 packet을 받는 장비가 아니라, **packet을 어디로 보낼지 판단하고 내부적으로 이동시킨 뒤 출력 링크로 전송하는 구조**

---

## 1-2. Data Plane과 Control Plane

- Router 내부 기능은 크게 **data plane**과 **control plane**으로 구분 가능

### Data Plane

- 실제 packet forwarding을 담당하는 부분
- hardware 중심으로 동작
- 매우 빠른 시간 단위에서 처리됨
- PPT 기준 nanosecond timeframe에서 동작

### Control Plane

- routing, management 기능을 담당하는 부분
- software 중심으로 동작
- data plane보다 느린 시간 단위에서 처리됨
- PPT 기준 millisecond timeframe에서 동작

⇒ Data plane은 **packet을 빠르게 전달하는 영역**, control plane은 **경로 계산과 관리 기능을 담당하는 영역**

---

# 2. Input Port Functions

## 2-1. Input Port

- **Input port** : router로 들어오는 packet을 가장 먼저 처리하는 부분
- 물리 계층, 링크 계층, 네트워크 계층 처리가 함께 일어남

Input port의 주요 기능은 다음과 같음

- **Line termination** : 물리 계층 수준의 bit 수신
- **Link layer protocol processing** : Ethernet 같은 link layer frame 처리
- **Lookup / forwarding** : forwarding table을 보고 output port 결정
- **Queueing** : switch fabric으로 보내는 속도보다 packet 도착 속도가 빠를 때 대기열 형성

⇒ Input port는 단순 입력 통로가 아니라, **packet 수신부터 forwarding 판단까지 수행하는 부분**

---

## 2-2. Decentralized Switching

- **Decentralized switching** : input port가 자체적으로 forwarding table을 사용해 output port를 결정하는 방식
- 각 input port가 자기 메모리에 forwarding table을 가지고 있음
- packet header field 값을 보고 forwarding table을 조회함
- 목표는 input port 처리를 **line speed**로 수행하는 것

### Line speed

- **Line speed** : packet이 들어오는 속도와 같은 속도로 처리하는 것
- input port 처리가 line speed보다 느리면 queueing 발생 가능

⇒ Decentralized switching은 **input port에서 빠르게 output port를 결정해 switch fabric으로 넘기는 방식**

---

# 3. Destination-based Forwarding과 Generalized Forwarding

## 3-1. Destination-based Forwarding

- **Destination-based forwarding** : destination IP address만 보고 packet을 어느 output port로 보낼지 결정하는 방식
- 전통적인 forwarding 방식
- forwarding table에는 목적지 주소 범위와 link interface가 저장됨

동작 방식은 다음과 같음

1. packet의 destination IP address 확인
2. forwarding table에서 해당 주소 범위 검색
3. 매칭되는 output link interface로 packet 전달

⇒ Destination-based forwarding은 **목적지 주소만 기준으로 forwarding하는 전통적 방식**

---

## 3-2. Generalized Forwarding

- **Generalized forwarding** : destination IP address뿐만 아니라 여러 header field 값을 기준으로 forwarding하는 방식
- match + action 구조와 연결됨
- packet header의 다양한 필드를 조건으로 사용할 수 있음

Destination-based forwarding과 차이는 다음과 같음

- Destination-based forwarding : destination IP address만 기준
- Generalized forwarding : 여러 header field 조합 기준

⇒ Generalized forwarding은 **더 다양한 조건으로 packet 처리 동작을 결정할 수 있는 forwarding 방식**

---

# 4. Longest Prefix Matching

## 4-1. Longest Prefix Matching

- **Longest prefix matching** : forwarding table에서 destination address와 매칭되는 prefix 중 가장 긴 prefix를 선택하는 방식
- 여러 entry가 동시에 매칭될 수 있을 때 가장 구체적인 entry를 선택함
- IP 주소 범위가 깔끔하게 나뉘지 않을 때 필요한 방식

예시 개념은 다음과 같음

- 어떤 destination address가 여러 prefix와 동시에 match될 수 있음
- 이때 가장 긴 prefix를 가진 entry 선택
- 더 긴 prefix는 더 구체적인 주소 범위를 의미함

⇒ Longest prefix matching은 **가장 구체적으로 일치하는 forwarding table entry를 선택하는 방식**

---

## 4-2. Longest Prefix Matching을 쓰는 이유

- IP 주소 범위가 항상 깔끔하게 나누어지지 않기 때문
- forwarding table entry 수를 줄이면서도 정확한 forwarding 가능
- 주소 범위 전체를 일일이 나열하지 않아도 됨

PPT와 책에서 강조하는 핵심은 다음과 같음

- Router는 destination address와 forwarding table의 prefix를 비교함
- 여러 prefix가 match되면 가장 긴 prefix 선택
- 선택된 prefix에 연결된 link interface로 packet 전달

⇒ Longest prefix matching은 **forwarding table을 효율적으로 관리하면서 정확한 출력 포트를 찾기 위한 방법**

---

## 4-3. TCAM

- **TCAM** : Ternary Content Addressable Memory
- Longest prefix matching을 빠르게 수행하기 위해 사용되는 메모리 구조
- 주소를 넣으면 table 크기와 관계없이 빠르게 검색 가능
- PPT에서는 one clock cycle 수준의 빠른 조회 가능성을 언급함

⇒ TCAM은 **router가 매우 빠른 속도로 forwarding table lookup을 수행하기 위한 하드웨어**

---

# 5. Switching Fabric

## 5-1. Switching Fabric

- **Switching fabric** : input port에서 output port로 packet을 이동시키는 router 내부 구조
- Router의 중심부 역할
- input link에서 들어온 packet을 적절한 output link로 전달함

### Switching rate

- **Switching rate** : packet을 input에서 output으로 이동시킬 수 있는 속도
- 보통 input/output line rate의 배수로 표현함
- N개의 input port가 있으면 이상적으로는 switching rate가 **N × line rate** 수준이면 좋음

⇒ Switching fabric은 **router 내부에서 packet이 지나가는 핵심 통로**

---

## 5-2. Switching Fabric의 세 가지 방식

Switching fabric의 대표 방식은 다음 세 가지임

1. **Switching via memory**
2. **Switching via bus**
3. **Switching via interconnection network**

⇒ 세 방식은 모두 input에서 output으로 packet을 옮기지만, 내부 전달 구조와 병목 지점이 다름

---

# 6. Switching via Memory

## 6-1. Switching via Memory

- **Switching via memory** : packet을 system memory에 복사한 뒤 output port로 보내는 방식
- 초기 router에서 사용된 방식
- 전통적인 컴퓨터 구조와 비슷함
- CPU가 switching을 직접 제어함

동작 방식은 다음과 같음

1. input port로 packet 도착
2. packet이 system memory로 복사됨
3. CPU가 output port를 결정함
4. packet이 memory에서 output port로 복사됨

⇒ Switching via memory는 **memory를 중간 저장소로 사용해 packet을 이동시키는 방식**

---

## 6-2. Switching via Memory의 한계

- packet 하나가 memory를 여러 번 거쳐야 함
- memory bandwidth에 의해 속도 제한 발생
- datagram 하나당 bus crossing이 여러 번 필요함
- 고속 router에는 부적합할 수 있음

⇒ Switching via memory의 병목 : **memory bandwidth**

---

# 7. Switching via Bus

## 7-1. Switching via Bus

- **Switching via bus** : input port에서 output port로 packet을 shared bus를 통해 전달하는 방식
- packet이 input port memory에서 output port memory로 bus를 통해 이동함
- memory 방식보다 단순하고 직접적인 구조

동작 방식은 다음과 같음

1. input port가 packet을 받음
2. packet이 shared bus를 통해 이동
3. 해당 output port가 packet 수신

⇒ Switching via bus는 **공유 bus를 통해 packet을 output port로 전달하는 방식**

---

## 7-2. Switching via Bus의 한계

- 여러 input port가 동시에 bus를 사용하려 하면 충돌 가능
- bus contention 발생
- switching speed가 bus bandwidth에 의해 제한됨

### Bus contention

- **Bus contention** : 여러 input port가 동시에 bus를 사용하려고 경쟁하는 상황
- bus는 한 번에 제한된 양만 처리 가능

⇒ Switching via bus의 병목 : **bus bandwidth**

---

# 8. Switching via Interconnection Network

## 8-1. Switching via Interconnection Network

- **Switching via interconnection network** : crossbar, Clos network 같은 내부 연결망을 사용해 packet을 이동시키는 방식
- 고성능 router에서 사용하는 방식
- 여러 packet을 병렬적으로 이동시킬 수 있음

대표 구조는 다음과 같음

- **Crossbar**
- **Clos network**
- **Multistage switch**
- **Multiple switching planes**

⇒ Interconnection network 방식은 **parallelism을 이용해 switching 성능을 높이는 방식**

---

## 8-2. Crossbar와 Multistage Switch

- **Crossbar** : input과 output을 격자 형태로 연결하는 switching 구조
- **Multistage switch** : 여러 단계의 작은 switch들을 조합해 큰 switch를 구성하는 방식
- 작은 switch들을 여러 단계로 연결해 n × n switching 구조 구현 가능

### Parallelism 활용

- datagram을 fixed length cell로 나눔
- cell들을 fabric 내부에서 병렬적으로 이동시킴
- 출구에서 다시 datagram으로 조립함

⇒ Multistage / interconnection 방식은 **작은 switching 요소들을 조합해 대규모 고속 switching을 가능하게 하는 방식**

---

## 8-3. Multiple Switching Planes

- **Multiple switching planes** : 여러 switching fabric plane을 병렬로 사용하는 구조
- switching capacity를 높이기 위한 방식
- PPT에서는 Cisco CRS router 예시 제시

Cisco CRS router 예시

- 기본 단위 : 8 switching planes
- 각 plane : 3-stage interconnection network
- switching capacity : 수백 Tbps 수준 가능

⇒ Multiple switching planes는 **병렬 fabric을 여러 개 사용해 router 성능을 확장하는 방식**

---

# 9. Input Port Queueing

## 9-1. Input Port Queueing

- **Input port queueing** : switch fabric으로 packet을 보내는 속도보다 input port에 packet이 들어오는 속도가 빠를 때 발생하는 queueing
- switch fabric이 충분히 빠르지 않으면 input port에서 packet이 대기함
- input buffer가 가득 차면 packet loss 발생 가능

⇒ Input port queueing은 **switch fabric 처리 속도가 입력 속도를 따라가지 못할 때 발생하는 대기 현상**

---

## 9-2. Head-of-the-Line Blocking

- **HOL blocking** : queue 맨 앞의 packet이 막혀서 뒤에 있는 packet들도 함께 이동하지 못하는 현상
- input queue에서 자주 설명되는 문제
- 앞 packet의 output port가 바쁘면, 뒤 packet이 다른 비어 있는 output port로 갈 수 있어도 기다려야 할 수 있음

예시 개념은 다음과 같음

- queue 앞 packet이 특정 output port로 가야 함
- 해당 output port가 이미 사용 중
- 뒤 packet은 다른 output port로 갈 수 있음
- 하지만 앞 packet 때문에 뒤 packet도 이동 불가

⇒ HOL blocking은 **앞 packet 하나가 뒤 packet들의 진행까지 막는 input queue 문제**

---

# 10. Output Port Queueing

## 10-1. Output Port Queueing

- **Output port queueing** : switch fabric에서 output port로 packet이 도착하는 속도가 output link 전송 속도보다 빠를 때 발생하는 queueing
- output port는 한 번에 하나의 link로만 packet을 전송 가능
- 여러 input port에서 같은 output port로 packet이 몰리면 queue가 생김

⇒ Output port queueing은 **output link가 packet 도착 속도를 따라가지 못할 때 발생하는 대기 현상**

---

## 10-2. Output Port Buffer Overflow

- Output port에는 packet을 임시 저장하는 buffer가 있음
- buffer가 가득 차면 더 이상 packet 저장 불가
- 이 경우 packet drop 발생 가능

### Packet loss 발생 원인

- congestion
- buffer 부족
- output link 속도보다 높은 arrival rate

⇒ Output port buffer overflow는 **혼잡으로 인해 packet이 저장되지 못하고 손실되는 상황**

---

# 11. Buffering

## 11-1. Buffering

- **Buffering** : packet을 즉시 전송하지 못할 때 임시로 저장하는 기능
- queueing delay와 packet loss를 조절하는 데 중요함
- 하지만 buffer가 많다고 항상 좋은 것은 아님

Buffer가 필요한 이유는 다음과 같음

- 순간적으로 packet이 몰리는 burst 흡수
- output link가 바쁠 때 packet 임시 저장
- packet drop 완화

⇒ Buffer는 **packet을 잠시 저장해 혼잡을 완화하는 공간**

---

## 11-2. How Much Buffering

- 전통적인 rule of thumb : 평균 buffer 크기 ≈ typical RTT × link capacity
- 예시 : RTT가 250ms이고 link capacity가 10Gbps이면 큰 buffer 필요
- 하지만 buffer가 너무 크면 queueing delay가 커질 수 있음

PPT의 핵심 내용은 다음과 같음

- 너무 적은 buffer : packet loss 증가
- 너무 많은 buffer : delay 증가
- 특히 home router에서 buffer가 너무 크면 real-time app 성능 저하 가능

⇒ Buffer 크기는 **packet loss와 delay 사이의 균형 문제**

---

## 11-3. Bufferbloat

- **Bufferbloat** : buffer가 너무 커서 packet이 오래 queue에 머물며 delay가 커지는 문제
- throughput은 유지될 수 있지만 delay가 매우 커질 수 있음
- real-time application에 악영향
- TCP 응답성도 느려질 수 있음

### Bufferbloat의 문제

- 긴 RTT 발생
- 실시간 서비스 성능 저하
- TCP 반응 지연
- queue가 불필요하게 오래 유지됨

⇒ Bufferbloat는 **buffer가 너무 커서 오히려 지연을 크게 만드는 문제**

---

# 12. Buffer Management

## 12-1. Buffer Management

- **Buffer management** : buffer가 가득 찼을 때 어떤 packet을 drop할지, 어떤 packet을 mark할지 결정하는 기능
- router의 queue 관리 정책과 연결됨

Buffer management의 주요 기능은 다음과 같음

- **Drop** : buffer가 가득 찼을 때 packet 폐기
- **Marking** : congestion 신호를 주기 위해 packet 표시

⇒ Buffer management는 **혼잡 상황에서 packet을 어떻게 처리할지 정하는 기능**

---

## 12-2. Drop Policy

- **Drop policy** : buffer가 가득 찼을 때 어떤 packet을 버릴지 결정하는 정책

대표 정책은 다음과 같음

- **Tail drop** : 새로 도착한 packet을 drop
- **Priority-based drop** : 우선순위에 따라 drop하거나 제거

### Tail Drop

- buffer가 꽉 찬 상태에서 새 packet이 도착하면 그 packet을 버림
- 가장 단순한 drop 방식

⇒ Tail drop은 **가득 찬 queue 뒤에 도착한 packet을 버리는 방식**

---

## 12-3. Marking

- **Marking** : packet을 버리지 않고 congestion 신호를 표시하는 방식
- ECN, RED 같은 방식과 관련 있음

### ECN

- **ECN** : Explicit Congestion Notification
- packet을 drop하지 않고 congestion을 알리는 표시 방식

### RED

- **RED** : Random Early Detection
- queue가 완전히 가득 차기 전에 미리 혼잡 신호를 주는 방식

⇒ Marking은 **packet 손실 없이 congestion을 미리 알리기 위한 방법**

---

# 13. Packet Scheduling

## 13-1. Packet Scheduling

- **Packet scheduling** : output link에서 다음에 어떤 packet을 보낼지 결정하는 기능
- output port queue에 여러 packet이 있을 때 전송 순서를 정함
- 네트워크 성능, 지연, 공정성에 직접 영향

대표 scheduling 방식은 다음과 같음

- FCFS / FIFO
- Priority scheduling
- Round Robin
- Weighted Fair Queueing

⇒ Packet scheduling은 **queue에 쌓인 packet들의 전송 순서를 정하는 정책**

---

# 14. FCFS / FIFO Scheduling

## 14-1. FCFS

- **FCFS** : First Come, First Served
- 먼저 도착한 packet을 먼저 전송하는 방식
- FIFO라고도 부름

### FIFO

- **FIFO** : First In, First Out
- queue에 먼저 들어온 packet이 먼저 나감

특징은 다음과 같음

- 구현이 단순함
- 도착 순서대로 처리함
- packet class나 priority를 따로 고려하지 않음

⇒ FCFS/FIFO는 **먼저 온 packet을 먼저 보내는 가장 단순한 scheduling 방식**

---

# 15. Priority Scheduling

## 15-1. Priority Scheduling

- **Priority scheduling** : packet을 class별로 분류하고, 높은 priority queue의 packet을 먼저 전송하는 방식
- traffic은 header field 등을 기준으로 분류 가능
- 각 priority class 내부에서는 FCFS 방식 사용 가능

동작 방식은 다음과 같음

1. packet arrival
2. header field 등을 기준으로 class 분류
3. high priority queue와 low priority queue에 저장
4. 가장 높은 priority queue에서 packet 선택
5. 같은 priority 안에서는 FCFS 처리

⇒ Priority scheduling은 **중요한 traffic을 먼저 처리할 수 있는 scheduling 방식**

---

## 15-2. Priority Scheduling의 의미

- 특정 traffic에 더 좋은 성능 제공 가능
- delay-sensitive traffic을 우선 처리 가능
- 하지만 낮은 priority traffic은 오래 기다릴 수 있음

### 비선점형 priority scheduling

- 낮은 priority packet이 이미 전송 중이면 중간에 끊지 않음
- 전송 중인 packet이 끝난 후 높은 priority packet 전송

⇒ Priority scheduling은 **성능 차등 제공이 가능하지만 공정성 문제가 생길 수 있는 방식**

---

# 16. Round Robin Scheduling

## 16-1. Round Robin

- **Round Robin scheduling** : 여러 class queue를 순서대로 돌아가며 packet을 하나씩 전송하는 방식
- traffic을 class별로 나눈 뒤 각 class queue를 순환적으로 확인함
- 해당 class에 packet이 있으면 하나 전송하고 다음 class로 넘어감

동작 방식은 다음과 같음

1. packet을 class별 queue에 저장
2. scheduler가 class queue를 순서대로 확인
3. packet이 있는 queue에서 하나 전송
4. 다음 queue로 이동
5. 이 과정 반복

⇒ Round Robin은 **여러 traffic class에 전송 기회를 돌아가며 제공하는 scheduling 방식**

---

# 17. Weighted Fair Queueing

## 17-1. WFQ

- **WFQ** : Weighted Fair Queueing
- Round Robin을 일반화한 방식
- 각 class에 weight를 부여함
- weight가 큰 class는 더 많은 service를 받음

### Weight

- **Weight** : 각 traffic class가 받을 service 비율
- 각 class i는 weight wi를 가짐
- 전체 weight 합에서 자기 weight만큼의 비율로 service를 받음

공식처럼 이해하면 다음과 같음

- class i의 service 비율 : `wi / Σwj`

⇒ WFQ는 **traffic class별 최소 bandwidth 보장과 공정성을 함께 고려하는 scheduling 방식**

---

# 18. Network Neutrality

## 18-1. Network Neutrality

- **Network neutrality** : ISP가 traffic을 어떻게 처리해야 하는지에 대한 기술적·사회적·경제적 원칙
- 기술적으로는 packet scheduling과 buffer management와 연결됨
- 어떤 traffic을 우선 처리할지, 어떤 traffic을 제한할지와 관련 있음

Network neutrality의 관점은 다음과 같음

- **Technical view** : ISP가 자원을 어떻게 공유하고 배분할지의 문제
- **Social / economic view** : 표현의 자유, 혁신, 경쟁 보호의 문제
- **Legal / policy view** : 법과 정책으로 규제할지의 문제

⇒ Network neutrality는 **packet 처리 정책이 공정성과 규제 문제로 이어지는 주제**

---

## 18-2. 2015 FCC Open Internet Order

2015년 미국 FCC의 Open Internet Order는 세 가지 원칙을 제시함

- **No blocking** : 합법적인 content, application, service, non-harmful device 차단 금지
- **No throttling** : 합법적인 Internet traffic을 content, application, service 기준으로 저하 금지
- **No paid prioritization** : 돈을 받고 특정 traffic을 우선 처리하는 행위 금지

⇒ 2015 FCC 원칙은 **차단 금지, 속도 저하 금지, 유료 우선처리 금지**

---

## 18-3. ISP 규제 관점

- ISP를 telecommunications service로 볼지, information service로 볼지가 규제 측면에서 중요함
- Telecommunications service로 보면 common carrier duties 적용 가능
- Information service로 보면 common carrier duties가 적용되지 않음

### Title II

- 통신 서비스에 적용
- reasonable rates, non-discrimination 등 의무 부과
- 규제 강함

### Title I

- 정보 서비스에 적용
- common carrier duties 없음
- 규제 약함

⇒ ISP의 법적 분류는 **망 중립성 규제 강도에 직접 영향**

---

# 19. 핵심 비교 정리

## 19-1. Data Plane vs Control Plane

- **Data plane** : packet forwarding 담당
- **Control plane** : routing, management 담당

차이는 다음과 같음

- Data plane : hardware 중심
- Control plane : software 중심
- Data plane : nanosecond 단위
- Control plane : millisecond 단위

⇒ Data plane은 **전달**, control plane은 **계산과 관리**

---

## 19-2. Destination-based Forwarding vs Generalized Forwarding

- **Destination-based forwarding** : destination IP address만 기준
- **Generalized forwarding** : 여러 header field 값 기준

⇒ Destination-based는 **전통적 방식**, generalized는 **더 유연한 match + action 방식**

---

## 19-3. Input Queueing vs Output Queueing

- **Input queueing** : switch fabric이 input 속도를 따라가지 못할 때 발생
- **Output queueing** : output link가 fabric에서 오는 packet 속도를 따라가지 못할 때 발생

차이는 다음과 같음

- Input queueing 원인 : switch fabric 속도 부족
- Output queueing 원인 : output link 속도 부족
- Input queueing 문제 : HOL blocking
- Output queueing 문제 : buffer overflow, packet loss

⇒ Input queueing은 **fabric 병목**, output queueing은 **output link 병목**

---

## 19-4. Switching 방식 비교

### Switching via Memory

- 중간에 memory 사용
- CPU 제어
- memory bandwidth 병목

### Switching via Bus

- shared bus 사용
- 구조 단순
- bus bandwidth 병목

### Switching via Interconnection Network

- crossbar, Clos, multistage network 사용
- 병렬 처리 가능
- 고성능 router에 적합

⇒ 고속 router일수록 **interconnection network 기반 switching 구조가 중요**

---

## 19-5. Scheduling 방식 비교

### FCFS / FIFO

- 도착 순서대로 전송
- 단순하지만 priority 고려 없음

### Priority Scheduling

- 높은 priority traffic 먼저 전송
- 성능 차등 가능
- 낮은 priority traffic 지연 가능

### Round Robin

- class별로 돌아가며 전송
- 각 class에 기회 제공

### WFQ

- weight 기반 공정 분배
- class별 bandwidth 보장 가능

⇒ Packet scheduling은 **단순 순서 처리부터 우선순위, 공정성, bandwidth 보장까지 확장되는 정책**