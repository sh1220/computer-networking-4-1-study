# Chapter 3

## Reliable Data Transfer의 핵심 목적

Reliable Data Transfer, 즉 **신뢰성 있는 데이터 전송**은 말 그대로 데이터를 보내는 쪽에서 받은 쪽까지 **손상 없이, 빠짐없이, 중복 없이, 순서대로 전달되도록 만드는 것**이다.

하지만 실제 네트워크는 완벽하지 않다.

패킷이 전송되는 동안 다음과 같은 문제가 생길 수 있다.

- 패킷 내부의 비트가 손상될 수 있음
- 패킷이 중간에 손실될 수 있음
- ACK가 손상되거나 손실될 수 있음
- 패킷이 너무 늦게 도착할 수 있음
- 중복 패킷이 도착할 수 있음
- 순서가 뒤바뀔 수 있음

따라서 전송 계층은 이런 불완전한 하위 계층 위에서 **마치 완벽한 채널처럼 보이게 하는 프로토콜**을 만들어야 한다.

---

## rdt의 기본 구조

rdt는 **reliable data transfer protocol**의 약자이다.

이 범위에서는 rdt를 단계적으로 발전시키면서 신뢰성 있는 전송을 어떻게 구현하는지 설명한다.

### rdt에서 사용하는 주요 인터페이스

- `rdt_send(data)`
    - 상위 계층, 즉 애플리케이션 계층이 전송 계층에게 데이터를 보내달라고 요청하는 함수
- `udt_send(packet)`
    - 전송 계층이 하위 계층, 즉 네트워크 계층으로 패킷을 내려보내는 함수
    - 여기서 `udt`는 unreliable data transfer를 의미함
    - 즉, 하위 계층은 신뢰할 수 없다고 가정함
- `rdt_rcv(packet)`
    - 수신 측 전송 계층이 하위 계층으로부터 패킷을 받는 함수
- `deliver_data(data)`
    - 수신 측 전송 계층이 데이터를 애플리케이션 계층으로 올려보내는 함수

핵심은 다음과 같다.

> 애플리케이션은 신뢰성 있는 전송을 원하지만, 실제 네트워크는 신뢰할 수 없기 때문에 전송 계층이 중간에서 오류 검출, 재전송, 순서 관리 등을 수행해야 한다.
> 

---

## FSM으로 rdt 설명하기

rdt 프로토콜은 **FSM, Finite State Machine**으로 설명된다.

FSM은 현재 상태와 발생한 이벤트에 따라 다음 상태와 수행할 행동이 결정되는 구조이다.

예를 들어 송신자는 다음과 같은 상태를 가질 수 있다.

- 상위 계층으로부터 데이터를 기다리는 상태
- ACK를 기다리는 상태

수신자는 다음과 같은 상태를 가질 수 있다.

- 하위 계층으로부터 패킷을 기다리는 상태
- 특정 sequence number의 패킷을 기다리는 상태

FSM에서 중요한 것은 다음이다.

- 현재 상태가 무엇인지
- 어떤 이벤트가 발생했는지
- 그 이벤트에 대해 어떤 행동을 하는지
- 다음 상태가 무엇인지

---

## rdt1.0: 완벽하게 신뢰적인 채널

rdt1.0은 가장 단순한 버전이다.

### 가정

하위 채널이 완벽하게 신뢰적이라고 가정한다.

즉,

- 비트 오류 없음
- 패킷 손실 없음
- 순서 뒤바뀜 없음

### 송신자 동작

송신자는 상위 계층에서 데이터를 받으면 패킷을 만들고 바로 보낸다.

```
packet = make_pkt(data)
udt_send(packet)
```

### 수신자 동작

수신자는 패킷을 받으면 데이터를 꺼내서 상위 계층으로 전달한다.

```
extract(packet, data)
deliver_data(data)
```

### rdt1.0의 특징

rdt1.0은 오류나 손실이 전혀 없다고 가정하기 때문에 별도의 복잡한 처리가 필요 없다.

ACK도 필요 없고, NAK도 필요 없고, 재전송도 필요 없다.

하지만 실제 네트워크에서는 비트 오류나 손실이 발생할 수 있으므로 rdt1.0은 현실적이지 않다.

---

## rdt2.0: 비트 오류가 있는 채널

rdt2.0에서는 하위 채널에서 **비트 오류**가 발생할 수 있다고 가정한다.

즉, 패킷이 도착하긴 하지만 내용이 손상될 수 있다.

### 해결해야 할 문제

수신자는 받은 패킷이 정상인지 손상되었는지 판단해야 한다.

이를 위해 사용하는 것이 **checksum**이다.

### checksum

checksum은 패킷 내용이 전송 중에 바뀌었는지 확인하기 위한 값이다.

송신자는 데이터를 기반으로 checksum을 계산해서 패킷에 넣고, 수신자는 받은 패킷을 다시 계산해서 checksum이 맞는지 확인한다.

- checksum이 맞으면 정상 패킷으로 판단
- checksum이 다르면 손상된 패킷으로 판단

---

## ACK와 NAK

rdt2.0에서는 수신자가 송신자에게 패킷 수신 결과를 알려준다.

### ACK

ACK는 Acknowledgement의 약자이다.

수신자가 패킷을 정상적으로 받았다는 뜻이다.

### NAK

NAK는 Negative Acknowledgement의 약자이다.

수신자가 패킷을 받았지만 손상되었다는 뜻이다.

### rdt2.0 동작

정상 상황에서는 다음과 같이 동작한다.

1. 송신자가 패킷을 보냄
2. 수신자가 checksum으로 오류 확인
3. 오류가 없으면 ACK 전송
4. 송신자는 ACK를 받고 다음 데이터를 보낼 준비를 함

오류 상황에서는 다음과 같이 동작한다.

1. 송신자가 패킷을 보냄
2. 수신자가 checksum으로 오류 확인
3. 오류가 있으면 NAK 전송
4. 송신자는 NAK를 받고 같은 패킷을 재전송함

---

## rdt2.0의 치명적인 문제

rdt2.0은 데이터 패킷의 오류는 처리할 수 있지만, **ACK나 NAK 자체가 손상되는 경우**를 처리하지 못한다.

예를 들어 송신자가 패킷을 보냈고 수신자가 ACK를 보냈다고 하자.

그런데 ACK가 손상되어 송신자에게 도착하면 송신자는 이런 상황을 구분할 수 없다.

- 수신자가 제대로 받았는데 ACK만 깨진 것인지
- 수신자가 못 받아서 NAK를 보냈는데 NAK가 깨진 것인지

즉, 송신자는 수신자의 실제 상태를 모른다.

여기서 단순히 재전송하면 문제가 생긴다.

이미 수신자가 정상 패킷을 받았는데 송신자가 같은 패킷을 다시 보내면 **중복 패킷**이 발생할 수 있다.

따라서 rdt2.0에는 중복 패킷을 구분할 방법이 필요하다.

---

## rdt2.1: sequence number 추가

rdt2.1은 rdt2.0의 문제를 해결하기 위해 **sequence number**를 추가한다.

sequence number는 패킷 번호이다.

수신자는 sequence number를 보고 이 패킷이 새로운 패킷인지, 이미 받은 패킷의 중복인지 구분할 수 있다.

### 왜 sequence number가 필요한가?

ACK나 NAK가 손상되면 송신자는 같은 패킷을 다시 보낼 수 있다.

이때 수신자가 sequence number를 확인하면 다음과 같이 판단할 수 있다.

- 기대하던 번호의 패킷이면 새 패킷으로 처리
- 이미 받은 번호의 패킷이면 중복 패킷으로 판단하고 버림

즉, sequence number는 **중복 패킷 감지용 번호**이다.

---

## rdt2.1에서 0과 1만으로 충분한 이유

rdt2.1은 stop-and-wait 방식이다.

stop-and-wait는 송신자가 한 번에 하나의 패킷만 보내고, 그 패킷에 대한 ACK를 받을 때까지 다음 패킷을 보내지 않는 방식이다.

따라서 동시에 여러 패킷이 날아다니지 않는다.

이 상황에서는 수신자가 구분해야 하는 것은 단순히 다음 둘뿐이다.

- 직전에 받은 패킷과 같은 패킷인가?
- 새 패킷인가?

그래서 sequence number는 0과 1, 두 개만 있어도 충분하다.

예를 들어 다음처럼 번갈아 사용한다.

```
pkt0 → ACK0 → pkt1 → ACK1 → pkt0 → ACK0 → pkt1 ...
```

---

## rdt2.1의 송신자와 수신자 역할

### 송신자

송신자는 다음을 기억해야 한다.

- 현재 보내야 할 패킷 번호가 0인지 1인지
- ACK 또는 NAK가 손상되었는지
- ACK가 오면 다음 패킷으로 넘어갈지
- NAK나 손상된 응답이 오면 현재 패킷을 재전송할지

### 수신자

수신자는 다음을 기억해야 한다.

- 현재 기대하는 패킷 번호가 0인지 1인지
- 받은 패킷이 손상되었는지
- 받은 패킷이 중복인지
- 정상 패킷이면 데이터를 상위 계층으로 전달할지
- 중복 패킷이면 데이터를 다시 전달하지 않고 ACK만 보낼지

중요한 점은 수신자는 자신의 마지막 ACK가 송신자에게 제대로 도착했는지 알 수 없다는 것이다.

따라서 중복 패킷을 받을 가능성을 항상 고려해야 한다.

---

## rdt2.2: NAK 없는 프로토콜

rdt2.2는 rdt2.1과 같은 기능을 하지만 **NAK를 사용하지 않는다.**

대신 ACK만 사용한다.

### 핵심 아이디어

수신자가 손상된 패킷을 받으면 NAK를 보내는 대신, **마지막으로 정상적으로 받은 패킷에 대한 ACK를 다시 보낸다.**

즉, 중복 ACK를 보낸다.

예를 들어 수신자가 pkt0까지 정상적으로 받았고, 다음으로 pkt1을 기대하고 있다고 하자.

그런데 pkt1이 손상되어 도착하면 수신자는 NAK를 보내는 대신 다시 ACK0을 보낸다.

송신자는 ACK0이 중복으로 온 것을 보고 pkt1에 문제가 있었다고 판단하고 pkt1을 재전송한다.

### rdt2.2의 의미

rdt2.2는 TCP와 연결되는 중요한 아이디어이다.

TCP도 기본적으로 NAK를 사용하지 않고 ACK 기반으로 동작한다.

즉, TCP는 명시적인 NAK 대신 **중복 ACK**를 통해 손실이나 오류를 추정한다.

---

## rdt3.0: 비트 오류와 패킷 손실이 있는 채널

rdt3.0에서는 하위 채널이 더 불안정하다고 가정한다.

이제는 다음 두 가지 문제가 모두 발생할 수 있다.

- 패킷이 손상될 수 있음
- 패킷 자체가 손실될 수 있음

패킷이 손실되면 수신자는 아무것도 받지 못한다.

그러면 ACK나 NAK도 보낼 수 없다.

따라서 송신자는 무한정 기다리면 안 된다.

---

## rdt3.0의 해결책: timer와 timeout

rdt3.0에서는 송신자가 패킷을 보낸 후 **타이머**를 시작한다.

일정 시간 안에 ACK가 오지 않으면 송신자는 패킷이 손실되었다고 판단하고 재전송한다.

### 동작 방식

1. 송신자가 패킷을 보냄
2. 송신자가 타이머 시작
3. ACK가 시간 안에 도착하면 다음 패킷 전송
4. ACK가 시간 안에 도착하지 않으면 timeout 발생
5. 송신자는 같은 패킷 재전송

여기서 중요한 점은 실제로 패킷이 손실된 것이 아니라 단순히 지연되었을 수도 있다는 것이다.

이 경우 재전송된 패킷은 중복 패킷이 될 수 있다.

하지만 sequence number가 있기 때문에 수신자는 중복 패킷을 구분해서 버릴 수 있다.

---

## rdt3.0이 처리하는 상황

rdt3.0은 다음 상황들을 처리할 수 있다.

### 정상 전송

- 송신자가 pkt0 전송
- 수신자가 pkt0 수신
- 수신자가 ACK0 전송
- 송신자가 ACK0 수신
- 다음 패킷 전송

### 데이터 패킷 손실

- 송신자가 pkt0 전송
- pkt0이 중간에서 손실됨
- 수신자는 아무것도 받지 못함
- ACK도 오지 않음
- 송신자 timeout 발생
- 송신자가 pkt0 재전송

### ACK 손실

- 수신자는 pkt0을 정상적으로 받음
- ACK0을 보냄
- ACK0이 중간에서 손실됨
- 송신자는 ACK를 받지 못해 timeout 발생
- 송신자가 pkt0 재전송
- 수신자는 pkt0이 중복임을 알고 데이터를 다시 올리지 않음
- 대신 ACK0을 다시 보냄

### premature timeout / delayed ACK

premature timeout은 ACK가 손실된 것이 아니라 늦게 도착했는데 송신자가 너무 빨리 timeout을 발생시키는 상황이다.

이 경우에도 송신자는 패킷을 재전송하고, 수신자는 sequence number로 중복을 감지한다.

---

## rdt3.0의 한계: Stop-and-Wait 성능 문제

rdt3.0은 신뢰성은 제공하지만 성능이 매우 나쁘다.

이유는 stop-and-wait 방식이기 때문이다.

stop-and-wait에서는 송신자가 패킷 하나를 보낸 뒤 ACK가 올 때까지 기다린다.

기다리는 동안 링크는 거의 놀고 있을 수 있다.

### 예시

조건이 다음과 같다고 하자.

- 링크 속도: 1 Gbps
- 전파 지연: 15 ms
- 패킷 크기: 8000 bits

패킷을 링크에 밀어 넣는 시간은 다음과 같다.

```
L / R = 8000 bits / 1 Gbps = 8 microseconds
```

왕복 시간 RTT는 대략 다음과 같다.

```
RTT = 30 ms
```

송신자 이용률은 다음과 같이 계산된다.

```
U_sender = (L/R) / (RTT + L/R)
```

값을 넣으면 약 0.027% 정도밖에 되지 않는다.

즉, 대부분의 시간 동안 송신자는 ACK를 기다리기만 한다.

결론적으로 rdt3.0은 신뢰성은 있지만 성능이 매우 낮다.

---

## Pipelining

### Pipelining의 필요성

stop-and-wait의 문제를 해결하기 위해 등장한 것이 **pipelining**이다.

pipelining은 ACK를 기다리지 않고 여러 개의 패킷을 연속으로 보내는 방식이다.

즉, 여러 패킷이 동시에 네트워크 안에 존재할 수 있다.

이를 **in-flight packet**이라고 한다.

### pipelining의 핵심

- 송신자가 여러 패킷을 연속으로 보낼 수 있음
- ACK를 기다리는 동안에도 다음 패킷을 보낼 수 있음
- 링크 이용률이 크게 증가함
- sequence number 범위가 더 커져야 함
- 송신자 또는 수신자 쪽에 buffering이 필요할 수 있음

pipelining을 구현하는 대표적인 방식은 다음 두 가지이다.

- Go-Back-N
- Selective Repeat

---

## Go-Back-N

Go-Back-N은 pipelining 방식 중 하나이다.

송신자는 최대 N개의 패킷을 ACK 없이 연속으로 보낼 수 있다.

이때 N을 window size라고 한다.

### sender window

송신자는 window 안에 있는 패킷만 보낼 수 있다.

예를 들어 window size가 4라면 송신자는 다음과 같이 보낼 수 있다.

```
pkt0, pkt1, pkt2, pkt3
```

이 4개에 대한 ACK가 오기 전까지는 다음 패킷을 더 보내지 못한다.

ACK가 오면 window가 앞으로 이동하고 새로운 패킷을 보낼 수 있다.

### Go-Back-N의 ACK 방식: cumulative ACK

Go-Back-N에서는 cumulative ACK를 사용한다.

cumulative ACK는 특정 번호까지의 모든 패킷을 정상적으로 받았다는 뜻이다.

예를 들어 `ACK 3`은 다음을 의미한다.

```
pkt0, pkt1, pkt2, pkt3까지 정상적으로 받았다.
```

즉, ACK가 하나의 패킷만 확인하는 것이 아니라 그 번호까지의 모든 패킷을 누적 확인한다.

### Go-Back-N 송신자 동작

Go-Back-N 송신자는 다음과 같이 동작한다.

- 최대 N개의 연속된 패킷을 보낼 수 있음
- 각 패킷에는 k-bit sequence number가 있음
- ACK(n)을 받으면 n번까지 모두 정상 수신되었다고 판단
- window를 n+1부터 시작하도록 앞으로 이동
- 가장 오래된 unACKed packet에 대해 타이머를 둠
- timeout이 발생하면 해당 패킷과 그 뒤의 모든 패킷을 재전송함

예를 들어 pkt2에 대해 timeout이 발생하면 다음을 재전송한다.

```
pkt2, pkt3, pkt4, pkt5 ...
```

그래서 이름이 Go-Back-N이다.

문제가 생긴 지점으로 돌아가서 그 뒤를 다시 보내는 방식이다.

### Go-Back-N 수신자 동작

Go-Back-N 수신자는 기본적으로 순서대로 도착한 패킷만 인정한다.

수신자는 다음을 기억한다.

- 현재까지 순서대로 받은 마지막 패킷 번호
- 또는 다음으로 기대하는 패킷 번호

수신자는 정상적으로 순서에 맞는 패킷을 받으면 ACK를 보낸다.

하지만 순서가 맞지 않는 패킷이 오면 보통 버린다.

예를 들어 pkt0, pkt1을 받았고 pkt2를 기다리고 있는데 pkt3이 먼저 도착하면 수신자는 pkt3을 버린다.

그리고 마지막으로 정상적으로 받은 pkt1에 대한 ACK를 다시 보낸다.

```
기대한 것: pkt2
도착한 것: pkt3
처리: pkt3 discard, ACK1 재전송
```

이런 ACK를 duplicate ACK라고 볼 수 있다.

### Go-Back-N 예시

window size가 4라고 하자.

송신자가 다음 패킷들을 보낸다.

```
pkt0, pkt1, pkt2, pkt3
```

수신자가 pkt0, pkt1은 정상적으로 받았지만 pkt2가 손실되고 pkt3이 도착했다고 하자.

수신자는 pkt3을 받을 수는 있지만, pkt2가 없기 때문에 순서가 깨진다.

그래서 pkt3을 버리고 ACK1을 다시 보낸다.

그 후 pkt4, pkt5가 도착해도 pkt2가 없으므로 모두 버리고 ACK1을 반복해서 보낸다.

송신자 쪽에서 pkt2 timeout이 발생하면 송신자는 다음을 다시 보낸다.

```
pkt2, pkt3, pkt4, pkt5
```

Go-Back-N의 단점은 이미 수신자에게 도착했을 수도 있는 뒤쪽 패킷들을 다시 보내야 한다는 것이다.

즉, 재전송 낭비가 생길 수 있다.

---

## Selective Repeat

Selective Repeat은 Go-Back-N보다 더 효율적인 pipelining 방식이다.

Go-Back-N은 손실된 패킷 이후의 모든 패킷을 다시 보내지만, Selective Repeat은 **손실된 패킷만 선택적으로 재전송**한다.

### Selective Repeat의 특징

- 송신자는 여러 패킷을 동시에 보낼 수 있음
- 수신자는 정상적으로 받은 패킷 각각에 대해 개별 ACK를 보냄
- 수신자는 순서가 맞지 않게 도착한 패킷도 버리지 않고 buffer에 저장할 수 있음
- 송신자는 각 unACKed packet에 대해 개별 timer를 가짐
- timeout이 발생한 패킷만 재전송함

### Selective Repeat 수신자 동작

Selective Repeat 수신자는 out-of-order 패킷도 저장할 수 있다.

예를 들어 pkt2가 손실되고 pkt3, pkt4, pkt5가 먼저 도착했다고 하자.

Go-Back-N에서는 pkt3, pkt4, pkt5를 버린다.

하지만 Selective Repeat에서는 다음과 같이 처리한다.

```
pkt3 수신 → buffer에 저장, ACK3 전송
pkt4 수신 → buffer에 저장, ACK4 전송
pkt5 수신 → buffer에 저장, ACK5 전송
```

이후 pkt2가 재전송되어 도착하면 수신자는 다음을 한 번에 상위 계층으로 전달할 수 있다.

```
pkt2, pkt3, pkt4, pkt5
```

이 방식은 불필요한 재전송을 줄여준다.

### Selective Repeat 송신자 동작

Selective Repeat 송신자는 다음과 같이 동작한다.

- 상위 계층에서 데이터가 오면 window 안에 남는 sequence number가 있을 때만 전송
- ACK(n)이 오면 n번 패킷을 정상 수신된 것으로 표시
- n이 sendbase라면 window를 다음 미확인 패킷까지 이동
- timeout(n)이 발생하면 n번 패킷만 재전송
- 각 패킷마다 개별 타이머가 필요함

즉, Go-Back-N보다 더 복잡하지만 효율적이다.

---

## Go-Back-N과 Selective Repeat 비교

| 구분 | Go-Back-N | Selective Repeat |
| --- | --- | --- |
| ACK 방식 | cumulative ACK | individual ACK |
| out-of-order 패킷 | 보통 버림 | buffer에 저장 |
| 재전송 범위 | 손실 패킷부터 뒤의 모든 패킷 | 손실된 패킷만 |
| 타이머 | 가장 오래된 unACKed packet 중심 | 각 패킷별 timer |
| 구현 난이도 | 상대적으로 단순 | 상대적으로 복잡 |
| 효율성 | 낮을 수 있음 | 높음 |
| 수신자 버퍼 | 적게 필요 | 많이 필요할 수 있음 |

---

## Selective Repeat의 sequence number 딜레마

Selective Repeat에서는 sequence number가 너무 작으면 문제가 생긴다.

예를 들어 sequence number가 0, 1, 2, 3만 있고 window size가 3이라고 하자.

그러면 sequence number가 너무 빨리 반복된다.

이때 수신자는 어떤 패킷이 새 패킷인지, 예전에 보냈던 패킷이 지연되어 온 것인지 구분하지 못할 수 있다.

즉, sequence number가 wrap around되면서 ambiguity가 생긴다.

### 해결 조건

Selective Repeat에서는 window size가 sequence number 공간의 절반 이하여야 한다.

```
window size ≤ sequence number space / 2
```

sequence number가 k비트라면 sequence number space는 다음과 같다.

```
2^k
```

따라서 Selective Repeat에서 안전한 window size는 다음과 같다.

```
N ≤ 2^(k-1)
```

예를 들어 sequence number가 2비트라면 가능한 번호는 0, 1, 2, 3 총 4개이다.

이 경우 window size는 최대 2여야 안전하다.

```
2-bit sequence number → sequence space = 4
safe window size ≤ 2
```

---

## rdt 버전별 요약

| 버전 | 가정 | 추가된 기능 | 한계 |
| --- | --- | --- | --- |
| rdt1.0 | 오류 없음, 손실 없음 | 단순 전송 | 현실적이지 않음 |
| rdt2.0 | 비트 오류 있음 | checksum, ACK, NAK, 재전송 | ACK/NAK 손상 처리 불가 |
| rdt2.1 | ACK/NAK 손상 가능 | sequence number 추가 | NAK 사용 |
| rdt2.2 | rdt2.1과 동일 | NAK 제거, 중복 ACK 사용 | 손실은 아직 처리 못함 |
| rdt3.0 | 오류 + 손실 가능 | timer, timeout, 재전송 | stop-and-wait라 성능 낮음 |
| pipelining | 여러 패킷 동시 전송 | window 사용 | 순서 관리와 buffering 필요 |
| Go-Back-N | pipelining | cumulative ACK | 손실 이후 패킷을 모두 재전송 |
| Selective Repeat | pipelining | individual ACK, buffering | 구현 복잡, sequence number 조건 필요 |

# Chapter 4

## Network Layer의 역할

네트워크 계층은 송신 호스트에서 수신 호스트까지 데이터를 전달하는 역할을 한다.

전송 계층에서 내려온 segment는 네트워크 계층에서 datagram으로 캡슐화된다.

### 송신 측

송신 측 네트워크 계층은 다음 일을 한다.

```
transport segment → network-layer datagram으로 캡슐화 → link layer로 전달
```

### 수신 측

수신 측 네트워크 계층은 다음 일을 한다.

```
datagram 수신 → transport segment 추출 → transport layer로 전달
```

## 라우터

라우터는 지나가는 모든 IP datagram의 header field를 검사한다.

그 후 datagram을 적절한 output port로 이동시켜 목적지까지 가는 경로를 따라 전달한다.

즉, 라우터는 end-to-end 경로 중간에서 패킷을 계속 다음 링크로 넘기는 장치이다.

---

## Network Layer의 두 핵심 기능

네트워크 계층의 가장 중요한 기능은 두 가지이다.

- forwarding
- routing

### Forwarding

Forwarding은 라우터 하나의 내부에서 일어나는 동작이다.

즉, 어떤 패킷이 라우터의 input link로 들어왔을 때, 이 패킷을 어떤 output link로 내보낼지 결정하는 과정이다.

쉽게 말하면 다음과 같다.

> 라우터 한 개 안에서 패킷을 입구에서 알맞은 출구로 보내는 것
> 

여행 비유로 설명하면 forwarding은 고속도로에서 하나의 분기점이나 교차로를 통과하는 것과 같다.

즉, 전체 여행 계획이 아니라 지금 이 지점에서 어느 방향으로 나갈지를 정하는 것이다.

### Routing

Routing은 출발지에서 목적지까지 패킷이 어떤 경로를 따라갈지 결정하는 과정이다.

즉, 네트워크 전체 관점에서 경로를 계산하는 일이다.

쉽게 말하면 다음과 같다.

> 출발지에서 목적지까지의 전체 길을 정하는 것
> 

여행 비유로 설명하면 routing은 전체 여행 경로를 계획하는 것이다.

예를 들어 서울에서 부산까지 가기 위해 어떤 고속도로를 탈지 정하는 것이 routing이다.

| 구분 | Forwarding | Routing |
| --- | --- | --- |
| 관점 | 라우터 하나 내부 | 네트워크 전체 |
| 역할 | input port로 들어온 패킷을 output port로 보냄 | 출발지부터 목적지까지 경로 결정 |
| 시간 규모 | 매우 빠름 | 상대적으로 느림 |
| 비유 | 교차로 하나 통과 | 전체 여행 계획 |
| 관련 영역 | data plane | control plane |

---

## Data Plane과 Control Plane

### Data Plane

Data plane은 각 라우터에서 로컬하게 수행되는 기능이다.

어떤 datagram이 라우터 input port에 도착했을 때, forwarding table을 보고 어떤 output port로 보낼지 결정한다.

Data plane은 실제 패킷을 빠르게 처리해야 하므로 매우 빠른 시간 단위로 동작한다.

핵심은 다음이다.

```
도착한 패킷의 header 확인 → forwarding table lookup → output port로 전달
```

### Control Plane

Control plane은 네트워크 전체 관점에서 경로를 결정하는 기능이다.

즉, datagram이 source host에서 destination host까지 어떤 라우터들을 거쳐 갈지 결정한다.

Control plane은 forwarding table을 만들거나 갱신하는 역할을 한다.

즉, data plane이 forwarding table을 사용한다면, control plane은 그 forwarding table이 만들어지도록 하는 역할을 한다.

| 구분 | Data Plane | Control Plane |
| --- | --- | --- |
| 기능 | 실제 패킷 forwarding | 경로 계산 및 forwarding table 생성 |
| 범위 | 라우터 내부의 local 기능 | 네트워크 전체의 logic |
| 시간 규모 | 빠름 | 상대적으로 느림 |
| 예시 | input port에서 output port 결정 | routing algorithm 수행 |
| 질문 | “이 패킷을 어느 포트로 보낼까?” | “목적지까지 어떤 경로가 좋을까?” |

---

## Control Plane의 두 가지 방식

### 1. Per-router control plane

전통적인 방식으로, 각 라우터 안에 routing algorithm이 들어 있고, 라우터들이 서로 정보를 교환하면서 forwarding table을 계산한다.

특징은 다음과 같다.

- 각 라우터가 control plane 기능을 가짐
- 라우터들이 서로 routing 정보를 교환함
- 각 라우터가 자신의 forwarding table을 계산함

즉, control logic이 네트워크 전체에 분산되어 있다.

### 2. SDN control plane

SDN은 Software-Defined Networking의 약자이다.

SDN에서는 control plane이 라우터 각각에 들어있는 것이 아니라, remote controller에 집중된다.

remote controller가 네트워크 전체를 보고 forwarding table을 계산한 뒤, 각 라우터에 설치한다.

특징은 다음과 같다.

- control logic이 중앙 controller에 있음
- 라우터는 주로 forwarding 기능을 수행함
- controller가 forwarding table을 계산하고 설치함
- 네트워크 관리와 정책 적용이 유연해짐

| 구분 | Per-router Control Plane | SDN Control Plane |
| --- | --- | --- |
| control logic 위치 | 각 라우터 내부 | remote controller |
| forwarding table 계산 | 각 라우터가 계산 | controller가 계산 |
| 구조 | 분산형 | 중앙집중형 |
| 라우터 역할 | routing + forwarding | 주로 forwarding |
| 장점 | 전통적이고 독립적 | 중앙 관리, 유연한 제어 |
| 단점 | 관리가 복잡할 수 있음 | controller 의존성 존재 |

---

## Network Service Model

Network service model은 네트워크 계층이 전송 계층에게 어떤 서비스를 제공하는지를 의미한다.

쉽게 말하면 다음 질문에 대한 답이다.

> 네트워크 계층은 sender에서 receiver까지 datagram을 운반할 때 어떤 보장을 해주는가?
> 

보장할 수 있는 서비스 예시는 다음과 같다.

### 개별 datagram에 대한 서비스

- guaranteed delivery
    - 패킷을 반드시 목적지까지 전달
- guaranteed delivery with less than 40 msec delay
    - 패킷을 반드시 전달하면서 지연 시간도 40ms 이하로 보장

### datagram flow에 대한 서비스

- in-order datagram delivery
    - 여러 datagram이 보낸 순서대로 도착하도록 보장
- guaranteed minimum bandwidth
    - 특정 flow에 최소 대역폭을 보장
- restrictions on inter-packet spacing
    - 패킷 간 간격 변화, 즉 jitter를 제한

---

## Internet의 service model: Best Effort

Internet의 기본 network service model은 **best effort**이다.

best effort는 말 그대로 “최선을 다해 전달하겠다”는 의미이다.

하지만 강한 보장은 하지 않는다.

Internet best-effort service는 다음을 보장하지 않는다.

- bandwidth 보장 없음
- packet loss 방지 보장 없음
- in-order delivery 보장 없음
- timing 보장 없음
- delay 보장 없음

즉, IP는 패킷을 목적지까지 보내려고 노력하지만, 반드시 도착한다고 보장하지 않는다.

그래서 TCP 같은 전송 계층 프로토콜이 신뢰성, 순서 보장, 흐름 제어, 혼잡 제어 등을 추가로 제공한다.

---

## 다양한 Network Service Model 비교

| Network Architecture | Service Model | Bandwidth | Loss | Order | Timing |
| --- | --- | --- | --- | --- | --- |
| Internet | Best Effort | 없음 | 보장 없음 | 보장 없음 | 보장 없음 |
| ATM | Constant Bit Rate | 일정 rate 보장 | 보장 | 순서 보장 | timing 보장 |
| ATM | Available Bit Rate | 최소 bandwidth 보장 | 보장 없음 | 순서 보장 | timing 보장 없음 |
| Internet | Intserv Guaranteed | 가능 | 보장 가능 | 순서 보장 | timing 보장 |
| Internet | Diffserv | 가능성 있음 | 가능성 있음 | 가능성 있음 | timing 보장 없음 |

> 핵심은 Internet의 기본 모델이 매우 단순한 best-effort라는 점 !!
> 

---

## Best-effort service가 성공한 이유

Internet의 best-effort service는 보장이 거의 없지만, 실제로는 매우 성공적인 모델이 되었다.

그 이유는 다음과 같다.

### 1. 구조가 단순하다

Best-effort는 복잡한 보장 기능을 네트워크 내부에 많이 넣지 않는다.

따라서 인터넷을 넓게 배포하고 확장하기 쉬웠다.

### 2. 충분한 대역폭 provision

네트워크 용량이 충분히 확보되면 실시간 애플리케이션도 대부분의 경우 쓸 만한 성능을 낼 수 있다.

예를 들어 음성 통화나 영상 스트리밍도 항상 완벽한 보장을 받는 것은 아니지만, 대부분의 시간 동안 충분히 잘 동작한다.

### 3. 애플리케이션 계층의 분산 서비스

데이터센터, CDN 같은 애플리케이션 계층의 분산 서비스가 클라이언트 가까이에 콘텐츠를 배치한다.

이렇게 하면 네트워크 계층이 강한 보장을 하지 않아도 사용자에게 좋은 성능을 제공할 수 있다.

### 4. 혼잡 제어

TCP 같은 전송 계층 프로토콜의 congestion control이 네트워크 혼잡을 줄이는 데 도움을 준다.

특히 elastic service, 즉 전송률을 조절할 수 있는 서비스는 네트워크 상태에 맞게 전송량을 조절할 수 있다.

---

## Network Layer 핵심 흐름

```
Transport segment
→ Network layer에서 datagram으로 캡슐화
→ 라우터들이 header를 보고 forwarding
→ 목적지 host의 network layer 도착
→ transport layer로 전달
```

## 핵심 구분

- Forwarding
    - 라우터 하나 내부에서 input port → output port로 보내는 동작
- Routing
    - source에서 destination까지 전체 경로를 결정하는 동작
- Data plane
    - 실제 패킷을 빠르게 forwarding하는 영역
- Control plane
    - forwarding table을 만들기 위해 경로를 계산하는 영역
- Best effort
    - Internet의 기본 service model
    - 전달을 시도하지만 bandwidth, loss, order, timing을 보장하지 않음
