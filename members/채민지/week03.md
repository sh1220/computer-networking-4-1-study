# Week 03 (03-17~03-23)
> - **Operation**: Compression 1
> - **Keyword**: Ch1 fundamentals (Edge/Core + delay intro)
> - **Deliverable**: Ch1 앞부분 핵심 개념맵 + delay 기본 정리
## 1) 범위
- Primary (8th 교재): Ch1 1.1 - 1.3 (p.31 - 64) + 1.4 일부 (p.65 - 71)
- v9 PPT: Ch1 slide 2 - 54
## 2) 핵심 요약 (5~10줄)
- 인터넷은 종단 시스템, 통신 링크, 패킷 스위치, ISP가 연결된 **network of networks**임
- 인터넷은 **구성요소 중심(nuts-and-bolts view)** 과 **서비스 중심(services view)** 두 관점에서 볼 수 있음
- 데이터는 한 번에 통째로 가는 것이 아니라 **packet** 단위로 쪼개져 전달됨
- 통신은 **protocol**에 의해 이루어지며, 메시지 형식·순서·동작 규칙을 포함함
- **network edge**에는 client, server, host가 위치함
- **access network**는 사용자를 첫 번째 router에 연결하는 구간이며 DSL, cable, FTTH, Ethernet, WiFi, 4G/5G 등이 있음
- **network core**에서는 router들이 packet switching 방식으로 패킷을 전달함
- 이때 핵심 개념은 **forwarding, routing, store-and-forward**임
- 성능은 **delay, loss, throughput**으로 설명할 수 있음
- delay는 processing, queueing, transmission, propagation delay로 나뉘며, 특히 queueing delay는 혼잡과 직접 연결됨

<details><summary> 요약 상세 정리
</summary>

- 인터넷은 단순히 컴퓨터 몇 대가 연결된 구조가 아닌 **종단 시스템(end system), 통신 링크(link), 패킷 스위치(router, switch)**, **여러 ISP**가 서로 연결된 networks of networks임 ⇒ 여러 네트워크가 이어져 전체 인터넷을 구성

1-1 ) 인터넷을 보는 두 개의 관점

1. nuts-and-bolts view(실제 구성요소 중심)

2. services view(웹, 이메일 같은 서비스 제공)

⇒ 두 관점을 함깨 봐야 인터넷 구조와 역할이 동시에 이해됨

- 인터넷에서 데이터는 **packet** 단위로 잘려서 전달됨(한 번에 통째로 X)
    - protocol(프로토콜) : 어떤 메시지를 보내는지, 어떤 순서로 주고받는지, 메시지 받았을 때 어떤 동작 하는지까지 포함하는 **통신 약속**
    
    → 네트워크 장치 사이엔 정해진 정차가 필요하며, 인터넷 통신 전반은 이런 프로토콜 위에서 동작
    
    ex. HTTP, TCP, IP, WiFi
    
- network edge(네트워크의 가장자리)엔 클라이언트와 서버 존재
    - **사용자**는 보통 **종단 시스템**(ex. PC, 노트북, 스마트폰)을 사용하고, **서버**는 **데이터 센터**에 위치하는 경우가 많음
    - access network : **종단 시스템이 인터넷에 처음 연결**되는 부분
        
        ex. DSL (전화선 이용), cable (케이블 TV 기반 HFC 구조 사용), FTTH (광섬유를 집까지 직접 연결하는 방식), Ethernet (학교나 회사 같은 기관망에서 많이 쓰임), WiFi & 4G/5G (무선 접속 방식)
        
        ⇒ **사용자의 기기가 첫 라우터까지 도달**하도록 해주는 **연결 구간**임
        
- network core :  인터넷의 중심부로, 여러 라우터가 연결된 구조임
    - **packet switching :** 각 패킷이 저장되었다가(store-and-forward) 다음 링크로 전달됨
        - 이 과정에서 라우터는 **forwarding**을 통해 패킷을 적절한 출력 링크로 보내고, 전체 경로는 **routing**에 의해 결정됨
        - 인터넷은 주로 packet switching을 사용함
        - 자원 공유가 효율적이지만, 혼잡 시 지연과 손실 발생 가능
    - circuit switching : 통신 전에 자원을 미리 예약하는 방식으로, 전통적인 전화망에 가까움
- 성능 측면에선 **delay, loss, throughput**가 중요함
    - 지연은 처리 지연, 큐잉 지연, 전송 지연, 전파 지연으로 나뉨
        - 큐잉 지원은 트래픽이 많아질수록 급격히 증가함
    - 라우터 버퍼가 가득 차면 패킷 손실이 발생함
    - throughput : 송신자에서 수신자까지 실제로 전당되는 속도
        - 전체 경로에서 가장 느린 링크가 병목이 되어 성능을 제한함
    
    ∴ 네트워크 성능은 단순히 빠른 링크 하나만으로 결정되지 않고, 전체 경로의 혼잡도와 구조에 영향을 받음
    
- 인터넷은 복잡한 시스템이기 때문에 layering으로 구조화함
    - 인터넷 프로토콜 스택은 보통 **application, transport, network, link, physical**의 5계층으로 설명함
    - 각 계층은 자신만의 기능을 수행하면서 아래 계층의 서비스를 사용함
    - 데이터를 보낼 때는 각 계층의 헤더가 붙음(**encapsulation**)
    
    ⇒ 이 구조 덕분에 복잡한 통신 시스템을 나눠 이해하고 설계할 수 있음
</details>

## 3 ) v8 vs v9 차이점 (최소 3개)

### 1. 9판에서는 content provider network를 인터넷 구조의 핵심 요소로 더 직접적으로 제시함

  - Google, Microsoft, Akamai 같은 콘텐츠 제공자가 자체 네트워크를 운영하며 사용자 가까이 서비스를 제공하는 구조가 슬라이드에서 분명하게 드러남

### 2. 9판에서는 SDN, cloud, smartphone, 4G/5G, 대규모 IoT device 증가 등 최신 인터넷 환경을 더 강하게 반영함

- SDN, cloud, smartphone 중심 인터넷, 4G/5G, 대규모 device 증가 같은 인터넷의 현재 확장 모습과 최근 변화를 역사/구조 설명 안에 직접 포함됨

### 3. 9판에서는 security를 Chapter 1의 독립적인 핵심 주제로 더 선명하게 다룸
- packet sniffing, IP spoofing, DoS, authentication, encryption, firewall 와 같은 공격/방어 개념을 초반부터 체계적으로 제시함
- v8 범위 정리에서는** edge/core와 delay 입문이 중심**인데, v9 PPT는 ** 같은 범위 안에서 인터넷 구조의 ‘현재성’** 까지 더 보여줌 ( v8은 기본 개념 이해 중심, v9는 기본 개념에 더해 현재 인터넷 생태계 변화까지 강조하는 느낌 )

## 4 ) 헷갈린 포인트 / 질문 (최소 2개)
- **protocol은 단순한 규칙과 정확히 무엇이 다를까?**
    
    → 단순 규칙이 아니라 메시지 형식, 순서, 예외 상황에서의 동작까지 포함하는지 ?
    
- **access network와 network core의 경계는 어디까지로 봐야 할까?**
    
    → 첫 번째 router까지를 access로 보는지, ISP 내부 일부까지 포함하는지 ?
    
- **queueing delay가 다른 delay보다 혼잡과 더 직접적으로 연결되는 이유는 무엇일?**
    
    → arrival rate가 transmission rate를 초과할 때 왜 급격히 증가하는지 ?

## 5 ) 발표 포인트 (1분 버전)
- 이번 범위는 인터넷의 가장 기본적인 큰 그림을 잡는 부분이었습니다. 인터넷은 하나의 네트워크가 아닌 여러 네트워크가 연결된 **network of networks** 이며, 데이터는 **packet** 단위로 전달됩니다. 사용자는 **access network** 을 통해 인터넷에 접속하고, **network edge** 에선 client와 server가 동작하며, **network core** 에서는 router가 packet switching 방식으로 패킷을 전달하는데, 이 과정에서 **forwarding**과 **routing**으로 핵심 역할을 합니다. 또한 성능을 이해하기 위해선 **delay, loss, throughput **개념이 중요하며, 특히 delay는 processing, queueing, transmission, propagation의 네 가지로 나눠 볼 수 있습니다.
## 6) 참고 링크