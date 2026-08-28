# 1. Network Layer Control Plane 개요

## 1-1. Control Plane

- **Control plane** : packet이 source에서 destination까지 어떤 경로를 따라갈지 결정하는 network layer 기능
- Network layer의 기능은 크게 **forwarding**과 **routing**으로 나눌 수 있음

### Forwarding

- **Forwarding** : router의 input으로 들어온 packet을 적절한 output port로 이동시키는 기능
- router 내부에서 수행되는 동작
- data plane의 핵심 기능

### Routing

- **Routing** : source에서 destination까지 packet이 지나갈 전체 경로를 결정하는 기능
- network 전체 관점에서 경로를 계산하는 과정
- control plane의 핵심 기능

⇒ Forwarding은 **router 내부의 packet 전달**, routing은 **network 전체 경로 결정**

---

## 1-2. Control Plane 구성 방식

Network control plane을 구성하는 방식은 크게 두 가지임

1. **Per-router control plane**
2. **SDN control plane**

⇒ 두 방식의 핵심 차이는 **경로 계산을 각 router가 직접 하느냐, 중앙 controller가 하느냐**

---

# 2. Per-router Control Plane

## 2-1. Per-router Control Plane

- **Per-router control plane** : 각 router가 자신의 routing algorithm component를 가지고 직접 경로 계산에 참여하는 방식
- 전통적인 routing 구조
- 각 router의 control plane이 서로 정보를 교환함
- 각 router가 자신의 forwarding table을 계산함

특징은 다음과 같음

- 모든 router 안에 routing algorithm 존재
- router끼리 control message 교환
- forwarding table이 각 router 내부에서 계산됨
- control plane 기능이 network 전체에 분산됨

⇒ Per-router control plane은 **각 router가 스스로 routing 정보를 계산하고 forwarding table을 만드는 분산형 구조**

---

# 3. SDN Control Plane

## 3-1. SDN Control Plane

- **SDN control plane** : logically centralized controller가 forwarding table을 계산하고 router에 설치하는 방식
- SDN은 **Software-Defined Networking**의 약자
- router 내부에서 직접 routing을 계산하기보다, remote controller가 계산을 담당함

특징은 다음과 같음

- remote controller가 forwarding table 계산
- 계산된 forwarding table을 router에 설치
- router는 주로 data plane 역할 수행
- control plane이 논리적으로 중앙집중화됨

⇒ SDN control plane은 **중앙 controller가 경로와 forwarding rule을 계산하고 router는 전달에 집중하는 구조**

---

## 3-2. Per-router Control Plane과 SDN Control Plane 차이

- **Per-router control plane** : 각 router가 routing algorithm을 직접 실행
- **SDN control plane** : remote controller가 forwarding table 계산 후 router에 설치

차이는 다음과 같음

- Per-router : control 기능이 각 router에 분산
- SDN : control 기능이 controller에 논리적으로 집중
- Per-router : router끼리 routing 정보 교환
- SDN : controller와 router의 control agent가 상호작용
- Per-router : 전통적 routing 구조
- SDN : programmable network 구조와 연결

⇒ Per-router는 **분산형 제어**, SDN은 **논리적 중앙집중형 제어**

---

# 4. Routing Protocols

## 4-1. Routing Protocol의 목표

- **Routing protocol** : sending host에서 receiving host까지 가는 좋은 경로를 결정하기 위한 protocol
- 여기서 좋은 경로는 상황에 따라 다르게 정의 가능

좋은 경로의 기준은 다음과 같음

- 가장 낮은 cost
- 가장 빠른 경로
- 가장 혼잡하지 않은 경로
- 운영자가 정의한 정책에 맞는 경로

### Path

- **Path** : packet이 source host에서 destination host까지 지나가는 router들의 순서

⇒ Routing protocol의 목표는 **router network를 통해 목적지까지 가는 좋은 path를 찾는 것**

---

# 5. Graph Abstraction

## 5-1. Graph Abstraction

- **Graph abstraction** : network를 graph 형태로 표현하는 방식
- routing algorithm을 설명하기 위해 router와 link를 graph로 추상화함

Graph는 다음과 같이 표현함

`G = (N, E)`

- **N** : node의 집합, 즉 router들의 집합
- **E** : edge의 집합, 즉 router 사이 link들의 집합

예시

- `N = {u, v, w, x, y, z}`
- `E = {(u,v), (u,x), (v,x), (v,w), (x,w), (x,y), (w,y), (w,z), (y,z)}`

⇒ Graph abstraction은 **router를 node로, link를 edge로 표현해 routing 문제를 계산 가능하게 만드는 방식**

---

## 5-2. Link Cost

- **Link cost** : 두 router를 직접 연결하는 link의 비용
- `c(a,b)` : node a와 node b 사이 direct link의 cost
- direct neighbor가 아니면 cost는 ∞로 표현함

예시

- `c(w,z) = 5`
- `c(u,z) = ∞`

Cost는 network operator가 정의할 수 있음

- 모든 link cost를 1로 설정
- bandwidth가 클수록 cost를 낮게 설정
- congestion이 심할수록 cost를 높게 설정

⇒ Link cost는 **routing algorithm이 좋은 경로를 계산할 때 사용하는 기준값**

---

# 6. Routing Algorithm 분류

## 6-1. Global vs Decentralized

Routing algorithm은 정보의 범위에 따라 두 가지로 나눌 수 있음

### Global Routing Algorithm

- 모든 router가 전체 network topology와 link cost 정보를 알고 있는 방식
- 대표 예시 : **Link-state algorithm**
- 모든 node가 같은 정보를 가지고 경로 계산

⇒ Global 방식은 **전체 지도를 알고 경로를 계산하는 방식**

### Decentralized Routing Algorithm

- 각 router가 처음에는 자신과 직접 연결된 neighbor의 link cost만 알고 시작하는 방식
- neighbor와 정보를 반복적으로 교환하면서 경로를 갱신함
- 대표 예시 : **Distance-vector algorithm**

⇒ Decentralized 방식은 **이웃과 정보를 주고받으며 점진적으로 경로를 계산하는 방식**

---

## 6-2. Static vs Dynamic

Routing algorithm은 route 변화 속도에 따라 static과 dynamic으로 나눌 수 있음

### Static Routing

- route가 느리게 변하거나 거의 고정된 방식
- 관리자가 수동으로 설정하는 경우가 많음

### Dynamic Routing

- route가 더 빠르게 변할 수 있는 방식
- 주기적 update 또는 link cost 변화에 반응하여 route 갱신

⇒ Static은 **거의 고정된 경로**, dynamic은 **network 변화에 따라 갱신되는 경로**

---

# 7. Link-State Routing Algorithm

## 7-1. Link-State Algorithm

- **Link-state algorithm** : 모든 node가 network topology와 link cost 정보를 알고 있는 상태에서 least-cost path를 계산하는 알고리즘
- centralized한 성격을 가짐
- 모든 node가 같은 link-state 정보를 가짐
- 대표 알고리즘 : **Dijkstra algorithm**

동작 전제는 다음과 같음

- 각 node가 전체 topology를 알고 있음
- 각 link의 cost를 알고 있음
- link-state broadcast를 통해 정보 공유
- 각 node는 자신을 source로 하여 모든 destination까지의 least-cost path 계산

⇒ Link-state algorithm은 **전체 network 정보를 바탕으로 최단 경로를 계산하는 방식**

---

## 7-2. Dijkstra Algorithm

- **Dijkstra algorithm** : 한 source node에서 모든 destination node까지의 least-cost path를 계산하는 알고리즘
- 반복적으로 가장 cost가 작은 node를 확정해 나가는 방식
- 계산 결과는 해당 source node의 forwarding table 생성에 사용됨

### 핵심 특징

- centralized algorithm
- 전체 topology와 link cost 필요
- source에서 모든 destination까지 least-cost path 계산
- k번 반복 후 k개의 destination에 대한 least-cost path 확정

⇒ Dijkstra algorithm은 **source 기준으로 모든 destination까지의 최소 비용 경로를 하나씩 확정하는 알고리즘**

---

## 7-3. Dijkstra Algorithm Notation

Dijkstra algorithm에서 사용하는 notation은 다음과 같음

- **c(x,y)** : node x에서 node y까지의 direct link cost
    - direct neighbor가 아니면 ∞
- **D(v)** : source에서 destination v까지의 현재 least-cost-path cost 추정값
- **p(v)** : source에서 v까지 가는 현재 경로에서 v 직전의 predecessor node
- **N'** : least-cost path가 확정된 node들의 집합

⇒ Dijkstra의 핵심은 **D(v)를 계속 갱신하면서 N'에 node를 하나씩 확정하는 것**

---

## 7-4. Dijkstra Algorithm 동작 순서

Dijkstra algorithm의 기본 흐름은 다음과 같음

1. Source node를 `N'`에 넣음
2. Source와 직접 연결된 neighbor의 cost를 D(v)에 초기화
3. 직접 연결되지 않은 node의 D(v)는 ∞로 설정
4. `N'`에 없는 node 중 D(v)가 가장 작은 node w 선택
5. w를 `N'`에 추가
6. w와 인접한 node들의 D(v)를 갱신
7. 모든 node가 `N'`에 들어갈 때까지 반복

갱신 공식은 다음과 같음

`D(v) = min(D(v), D(w) + c(w,v))`

의미는 다음과 같음

- 기존에 알고 있던 v까지의 비용
- source에서 w까지의 확정 비용 + w에서 v까지의 link cost
- 둘 중 더 작은 값을 선택

⇒ Dijkstra는 **가장 가까운 node를 확정하고, 그 node를 거쳐 가는 경로가 더 싼지 확인하는 방식**

---

# 8. Dijkstra Algorithm Example

## 8-1. 초기화

예시 graph에서 source를 `u`로 두고 least-cost path를 계산함

초기 상태는 다음과 같음

- `N' = {u}`
- u와 직접 연결된 node는 direct cost로 초기화
- 직접 연결되지 않은 node는 ∞

초기 D 값 예시

- `D(v) = 2, p(v) = u`
- `D(w) = 5, p(w) = u`
- `D(x) = 1, p(x) = u`
- `D(y) = ∞`
- `D(z) = ∞`

⇒ 초기에는 **source와 직접 연결된 neighbor의 cost만 알고 있는 상태**

---

## 8-2. 반복 과정

Dijkstra algorithm은 매 단계에서 `N'`에 없는 node 중 D 값이 가장 작은 node를 선택함

예시 흐름은 다음과 같음

1. `x` 선택
    - `D(x)=1`이 가장 작으므로 x 확정
    - x를 거쳐 v, w, y까지의 비용 갱신
2. `y` 선택
    - x를 통해 `D(y)=2`로 갱신됨
    - y를 거쳐 w, z 비용 갱신
3. `v` 선택
    - `D(v)=2` 확정
    - v를 거쳐 w로 가는 비용 확인
4. `w` 선택
    - `D(w)=3` 확정
    - w를 거쳐 z로 가는 비용 확인
5. `z` 선택
    - `D(z)=4` 확정

최종 확정 순서는 다음과 같음

`u → x → y → v → w → z`

⇒ Dijkstra는 **현재까지 가장 비용이 작은 node를 하나씩 확정하면서 전체 최단 경로 tree를 완성**

---

## 8-3. Least-cost-path Tree와 Forwarding Table

Dijkstra algorithm이 끝나면 source u 기준의 least-cost-path tree 생성 가능

예시 결과

- u에서 v : 직접 u-v link 사용
- u에서 x : 직접 u-x link 사용
- u에서 y : u-x-y 경로 사용
- u에서 w : u-x-y-w 경로 사용
- u에서 z : u-x-y-z 경로 사용

u의 forwarding table은 다음과 같이 만들어짐

| Destination | Outgoing link |
| --- | --- |
| v | (u, v) |
| x | (u, x) |
| y | (u, x) |
| w | (u, x) |
| z | (u, x) |

⇒ Forwarding table에는 **최종 목적지까지의 전체 경로가 아니라, 다음에 나갈 outgoing link가 저장됨**

---

## 8-4. Tie

- **Tie** : Dijkstra algorithm에서 같은 cost를 가진 후보 node가 여러 개 존재하는 상황
- tie가 발생하면 임의로 선택 가능
- 선택 방식에 따라 least-cost-path tree 모양은 달라질 수 있음
- 하지만 cost가 같은 경로라면 최소 비용이라는 점은 유지됨

⇒ Dijkstra에서 tie는 **같은 비용의 경로 중 하나를 선택하는 문제**

---

# 9. Dijkstra Algorithm 복잡도와 한계

## 9-1. Algorithm Complexity

Dijkstra algorithm의 복잡도는 node 수를 n이라고 할 때 다음과 같음

- 매 iteration마다 `N'`에 없는 node 중 최소 D 값을 찾아야 함
- 총 n번 반복
- 단순 구현에서는 `n(n+1)/2`번 비교 필요

복잡도는 다음과 같음

- 기본 구현 : `O(n²)`
- 더 효율적인 구현 : `O(n log n)` 가능

⇒ Dijkstra algorithm은 **기본 구현 기준 O(n²) 복잡도**

---

## 9-2. Message Complexity

Link-state 방식에서는 각 router가 자신의 link-state 정보를 모든 router에게 알려야 함

특징은 다음과 같음

- 각 router가 link-state information broadcast
- 한 source의 broadcast message를 퍼뜨리는 데 O(n) link crossing 가능
- 전체적으로 message complexity는 `O(n²)` 수준

⇒ Link-state routing은 **전체 topology 공유를 위해 많은 link-state message가 필요**

---

## 9-3. Route Oscillation

- **Route oscillation** : route가 안정적으로 고정되지 않고 계속 바뀌는 현상
- link cost가 traffic volume에 따라 변할 때 발생 가능
- traffic이 몰리면 cost가 증가하고, cost가 증가하면 route가 바뀌며, 다시 traffic 분포가 바뀜

문제 흐름은 다음과 같음

1. 특정 route에 traffic이 몰림
2. 해당 link cost 증가
3. routing algorithm이 다른 route 선택
4. traffic이 다른 route로 이동
5. 새로운 route의 cost 증가
6. 다시 route 변경

⇒ Link cost가 traffic load에 따라 변하면 **routing 결과가 계속 흔들리는 oscillation 발생 가능**

---

# 10. Distance Vector Routing Algorithm

## 10-1. Distance Vector Algorithm

- **Distance vector algorithm** : 각 node가 neighbor와 distance vector를 교환하며 least-cost path를 계산하는 알고리즘
- decentralized routing algorithm
- Bellman-Ford equation 기반
- 각 node는 전체 topology를 알 필요 없음
- neighbor의 정보만 받아 자신의 distance vector를 갱신함

⇒ Distance vector algorithm은 **이웃의 거리 정보를 이용해 분산적으로 최단 경로를 계산하는 방식**

---

## 10-2. Distance Vector

- **Distance vector** : 한 node가 모든 destination까지 가는 현재 추정 cost의 목록
- 각 node x는 모든 destination y에 대해 `Dₓ(y)` 값을 가짐

예시

- `Dₓ(y)` : node x에서 destination y까지의 least-cost path cost 추정값

초기 상태에서는 다음만 알고 있음

- 자기 자신까지의 cost = 0
- 직접 연결된 neighbor까지의 cost = link cost
- 모르는 destination까지의 cost = ∞

⇒ Distance vector는 **각 destination까지의 현재 최단 거리 추정표**

---

## 10-3. Bellman-Ford Equation

Distance vector algorithm은 Bellman-Ford equation을 사용함

공식은 다음과 같음

`Dₓ(y) = minᵥ { c(x,v) + Dᵥ(y) }`

의미는 다음과 같음

- x에서 y까지 가는 최소 비용은
- x의 neighbor v를 하나 선택해서
- x에서 v까지의 직접 비용 `c(x,v)`와
- v에서 y까지의 추정 비용 `Dᵥ(y)`를 더한 값 중 최솟값

여기서 v는 x의 모든 neighbor를 의미함

⇒ Bellman-Ford equation은 **이웃을 거쳐 목적지로 가는 비용 중 가장 작은 값을 선택하는 식**

---

## 10-4. Bellman-Ford Example

예시에서 u의 neighbor가 v, x, w이고 destination이 z인 경우

주어진 값

- `Dᵥ(z) = 5`
- `Dₓ(z) = 3`
- `D𝓌(z) = 3`
- `c(u,v) = 2`
- `c(u,x) = 1`
- `c(u,w) = 5`

계산은 다음과 같음

`Dᵤ(z) = min { c(u,v)+Dᵥ(z), c(u,x)+Dₓ(z), c(u,w)+D𝓌(z) }`

`Dᵤ(z) = min { 2+5, 1+3, 5+3 }`

`Dᵤ(z) = min { 7, 4, 8 } = 4`

따라서 next hop은 x

⇒ 최소값을 만드는 neighbor가 **해당 destination으로 가는 next hop**

---

# 11. Distance Vector Algorithm 동작 방식

## 11-1. 핵심 아이디어

Distance vector algorithm의 핵심 아이디어는 다음과 같음

- 각 node는 때때로 자신의 distance vector estimate를 neighbor에게 보냄
- neighbor로부터 distance vector를 받으면 Bellman-Ford equation으로 자신의 값을 갱신함
- 자신의 distance vector가 바뀌면 neighbor에게 다시 알림
- 더 이상 변경이 없으면 message를 보내지 않음

⇒ DV algorithm은 **이웃과 반복적으로 정보를 교환하면서 실제 least-cost path로 수렴하는 방식**

---

## 11-2. Iterative

- **Iterative** : 한 번에 끝나는 것이 아니라 여러 번 반복하면서 값이 갱신되는 성질
- 각 node는 neighbor의 DV를 받고 자신의 DV를 다시 계산함
- 계산 결과가 바뀌면 다시 neighbor에게 전파함

⇒ Distance vector는 **반복 계산을 통해 점점 정확한 경로 cost로 수렴**

---

## 11-3. Asynchronous

- **Asynchronous** : 모든 node가 동시에 동작할 필요 없는 성질
- 각 node는 다음 사건이 발생할 때만 동작함

동작을 유발하는 사건은 다음과 같음

- local link cost 변화
- neighbor로부터 DV update message 수신

⇒ DV algorithm은 **각 node가 독립적으로 update를 수행하는 비동기 방식**

---

## 11-4. Distributed

- **Distributed** : 중앙에서 전체를 계산하지 않고 각 node가 자기 정보를 바탕으로 계산하는 성질
- 각 node는 neighbor와만 정보를 교환함
- 전체 topology를 알 필요 없음

⇒ DV algorithm은 **각 router가 neighbor 정보만으로 경로를 계산하는 분산 알고리즘**

---

## 11-5. Self-stopping

- **Self-stopping** : 더 이상 값이 바뀌지 않으면 update message가 멈추는 성질
- node의 DV가 바뀐 경우에만 neighbor에게 알림
- 변화가 없으면 아무 동작도 하지 않음

⇒ DV algorithm은 **변경이 없으면 자연스럽게 update가 멈추는 방식**

---

# 12. Distance Vector Example

## 12-1. 초기 상태 t=0

초기 상태에서는 모든 node가 자기 neighbor까지의 distance만 알고 있음

예시에서 node a의 초기 DV는 다음과 같음

- `Dₐ(a) = 0`
- `Dₐ(b) = 8`
- `Dₐ(d) = 1`
- 나머지 destination = ∞

초기 특징은 다음과 같음

- 직접 연결된 neighbor만 알고 있음
- 직접 연결되지 않은 node는 ∞
- 모든 node가 자신의 local distance vector를 neighbor에게 보냄

⇒ t=0에서는 **각 node가 자기 주변 link cost만 알고 있는 상태**

---

## 12-2. Iteration 과정

각 iteration에서 모든 node는 다음 과정을 수행함

1. neighbor로부터 distance vector 수신
2. Bellman-Ford equation으로 자기 distance vector 계산
3. 값이 바뀌면 새로운 distance vector를 neighbor에게 전송

예시 흐름

- t=1 : neighbor의 DV를 받아 1-hop 너머 정보 반영
- t=2 : 정보가 더 멀리 전파되어 2-hop 너머 정보 반영
- 이후 반복 : network 전체로 정보 확산

⇒ DV example은 **neighbor 정보가 반복 계산을 통해 network 전체로 퍼지는 과정**

---

## 12-3. Node b의 계산 예시

t=1에서 b는 neighbor a, c, e로부터 DV를 받음

b의 neighbor는 다음과 같음

- a
- c
- e

b는 각 destination에 대해 다음처럼 계산함

`Dᵦ(y) = min { c(b,a)+Dₐ(y), c(b,c)+D𝚌(y), c(b,e)+Dₑ(y) }`

예시 결과는 다음과 같음

- `Dᵦ(a) = 8`
- `Dᵦ(c) = 1`
- `Dᵦ(d) = 2`
- `Dᵦ(e) = 1`
- `Dᵦ(f) = 2`
- `Dᵦ(h) = 2`
- `Dᵦ(g) = ∞`
- `Dᵦ(i) = ∞`

⇒ b는 **a, c, e를 거쳐 각 destination으로 가는 비용 중 최솟값을 선택**

---

## 12-4. State Information Diffusion

- **State information diffusion** : 한 node의 상태 정보가 반복적인 communication과 computation을 통해 network 전체로 퍼지는 과정
- DV algorithm에서는 정보가 한 번에 전체로 퍼지지 않음
- hop 단위로 점점 멀리 전파됨

예시

- t=0 : c의 상태 정보는 c에만 존재
- t=1 : c의 정보가 neighbor b에 영향
- t=2 : c의 정보가 b를 거쳐 a, e에도 영향
- t=3 : d, f, h까지 영향
- t=4 : g, i까지 영향

⇒ Distance vector에서는 **정보가 neighbor를 통해 hop-by-hop으로 확산**

---

# 13. Distance Vector: Link Cost Changes

## 13-1. Link Cost Change

DV algorithm에서 link cost가 바뀌면 node는 다음처럼 동작함

1. local link cost 변화 감지
2. routing information update
3. local distance vector 재계산
4. DV가 바뀌면 neighbor에게 알림

⇒ DV algorithm은 **local 변화가 neighbor update를 통해 network 전체로 전파되는 구조**

---

## 13-2. Good News Travels Fast

- **Good news travels fast** : link cost가 낮아지는 좋은 변화는 빠르게 전파되는 현상
- 더 좋은 경로가 생기면 neighbor들이 빠르게 새 cost를 계산함

예시 흐름

- t0 : y가 link cost 감소를 감지하고 DV 갱신 후 neighbor에게 알림
- t1 : z가 y의 update를 받고 자신의 DV 갱신
- t2 : y가 z의 update를 받지만 cost 변화가 없으므로 추가 message 없음

⇒ 좋은 소식은 **더 낮은 cost가 빠르게 반영되기 때문에 빠르게 수렴**

---

## 13-3. Bad News Travels Slow

- **Bad news travels slow** : link cost가 증가하거나 link가 나빠지는 정보는 느리게 전파되는 현상
- 잘못된 우회 경로를 서로 믿으면서 cost가 조금씩 증가할 수 있음
- 이 문제가 **count-to-infinity problem**

⇒ 나쁜 소식은 **router들이 서로의 오래된 정보를 믿으면서 천천히 반영될 수 있음**

---

## 13-4. Count-to-Infinity Problem

- **Count-to-infinity problem** : link cost 증가 또는 장애 상황에서 distance estimate가 조금씩 증가하며 수렴이 늦어지는 문제
- 두 node가 서로를 통해 목적지에 갈 수 있다고 착각하면서 발생함

예시 흐름

1. y에서 x로 가는 direct link cost가 60으로 증가
2. z는 이전에 y를 통해 x까지 cost 5라고 알고 있음
3. y는 z를 통해 x까지 갈 수 있다고 생각하고 cost 6으로 갱신
4. z는 y를 통해 cost 7이라고 갱신
5. y는 다시 z를 통해 cost 8이라고 갱신
6. 이런 식으로 cost가 계속 증가

⇒ Count-to-infinity는 **잘못된 경로 정보가 서로를 통해 반복 갱신되며 수렴이 느려지는 DV의 대표 문제**

---

## 13-5. Poisoned Reverse

- **Poisoned reverse** : count-to-infinity 문제를 완화하기 위한 방법
- 어떤 node가 destination x로 가는 경로를 neighbor z를 통해 사용한다면, z에게는 자신이 x까지 갈 수 없다고 알려주는 방식
- 즉, 해당 neighbor에게 cost를 ∞로 광고함

예시 개념

- y가 x로 가기 위해 z를 next hop으로 사용
- y는 z에게 `Dᵧ(x) = ∞`라고 알림
- z가 y를 통해 x로 가는 잘못된 loop를 만들지 못하게 함

⇒ Poisoned reverse는 **자신이 의존하는 neighbor에게는 해당 목적지로 못 간다고 알려 routing loop를 줄이는 방법**

---

# 14. Link-State와 Distance Vector 비교

## 14-1. Message Complexity

### Link-State

- 모든 router가 link-state information을 전체 network에 broadcast해야 함
- n개의 router가 있을 때 message complexity는 대략 `O(n²)`

### Distance Vector

- neighbor 사이에서만 distance vector 교환
- 전체 message 수는 topology와 convergence 과정에 따라 달라짐

⇒ LS는 **전체 공유라 message가 많고**, DV는 **neighbor 교환이라 구조는 단순하지만 수렴 시간 변동 가능**

---

## 14-2. Speed of Convergence

### Link-State

- Dijkstra algorithm 계산 복잡도 : `O(n²)`
- message complexity도 `O(n²)` 수준
- link cost가 traffic에 따라 변하면 oscillation 가능

### Distance Vector

- convergence time이 상황에 따라 달라짐
- routing loop 가능
- count-to-infinity 문제 가능

⇒ LS는 **빠르게 전체 정보를 계산할 수 있지만 oscillation 가능**, DV는 **분산적이지만 수렴이 느려질 수 있음**

---

## 14-3. Robustness

### Link-State

- router가 잘못된 link cost를 광고할 수 있음
- 하지만 각 router는 자기 forwarding table을 직접 계산함
- 오류 영향이 상대적으로 제한될 수 있음

### Distance Vector

- router가 잘못된 path cost를 광고할 수 있음
- 예를 들어 “모든 곳에 매우 낮은 cost로 갈 수 있음”이라고 광고 가능
- 다른 router들이 그 DV를 사용하므로 error가 network로 전파될 수 있음
- black-holing 가능

⇒ DV는 **다른 router의 정보를 직접 경로 계산에 사용하기 때문에 오류 전파 위험이 더 큼**

---

# 15. 핵심 비교 정리

## 15-1. Forwarding vs Routing

- **Forwarding** : router input에서 output으로 packet을 보내는 local 동작
- **Routing** : source에서 destination까지의 path를 결정하는 network-wide 과정

⇒ Forwarding은 **전달**, routing은 **경로 결정**

---

## 15-2. Per-router vs SDN Control Plane

- **Per-router control plane** : 각 router가 routing algorithm 실행
- **SDN control plane** : remote controller가 forwarding table 계산 및 설치

⇒ Per-router는 **분산형 제어**, SDN은 **논리적 중앙집중형 제어**

---

## 15-3. Link-State vs Distance Vector

### Link-State

- 전체 topology와 link cost 사용
- 모든 router가 같은 정보 보유
- Dijkstra algorithm 사용
- 전체 network 정보를 바탕으로 계산

### Distance Vector

- neighbor 정보만 사용
- Bellman-Ford equation 사용
- neighbor와 DV 반복 교환
- 분산적이고 비동기적으로 동작

⇒ Link-state는 **전체 지도를 보고 계산**, distance vector는 **이웃에게 들은 거리 정보로 계산**

---

## 15-4. Dijkstra vs Bellman-Ford

### Dijkstra

- Link-state algorithm에서 사용
- source에서 모든 destination까지 least-cost path 계산
- `D(v)`를 갱신하면서 가장 가까운 node부터 확정
- 전체 topology 필요

### Bellman-Ford

- Distance-vector algorithm에서 사용
- neighbor의 distance vector를 이용해 cost 계산
- `Dₓ(y) = minᵥ { c(x,v) + Dᵥ(y) }`
- 전체 topology 불필요

⇒ Dijkstra는 **전체 정보 기반 최단 경로 계산**, Bellman-Ford는 **neighbor 정보 기반 최단 경로 추정**

---

## 15-5. Good News vs Bad News

### Good News

- link cost 감소
- 더 좋은 경로 발견
- 빠르게 전파
- 빠르게 수렴

### Bad News

- link cost 증가
- 기존 경로가 나빠짐
- 잘못된 우회 정보 가능
- count-to-infinity 발생 가능

⇒ DV에서는 **좋은 소식은 빠르지만 나쁜 소식은 느리게 전파될 수 있음**