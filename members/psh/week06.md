| 주차 | 기간 | Primary(v8) 교재 범위 | v9 PPT 범위 |
|---|---|---|---|
| 6주차 | 04-07 ~ 04-13 | Ch2 2.5~2.7 (P2P, 비디오·CDN, 소켓), Ch3 3.1 (전송 계층 소개) | (강의 자료 기준으로 보완) |

원본 노트: [Week06 (04-07~04-13) · P2P / CDN / Socket / Transport intro](https://www.notion.so/Week06-04-07-04-13-P2P-CDN-Socket-Transport-intro-34177ca3c78281ba9461dbd0978fc79e)

# 2.5 Peer-to-Peer File Distribution
## P2P 아키텍처 개요
- 기존 애플리케이션(Web, Email, DNS)은 항상 가동 중인 인프라 서버에 의존하는 클라이언트-서버 아키텍처(client-server architecture)를 채택
- P2P 아키텍처(P2P architecture)는 항상 가동되는 서버 없이, 간헐적으로 연결되는 호스트 쌍(피어, peers)이 직접 통신
- 피어는 서비스 제공업체 소유가 아닌 사용자가 직접 제어하는 PC, 노트북, 스마트폰
- P2P 파일 배포: 단일 서버에서 다수의 피어로 대용량 파일을 배포 (예: Linux OS, 소프트웨어 패치, 비디오 파일)
- 클라이언트-서버 방식: 서버가 각 피어에게 파일 사본을 전송 → 서버에 막대한 부담, 많은 대역폭 소모
- P2P 방식: 각 피어가 받은 파일의 일부를 다른 피어에게 재배포 → 서버 부담 경감
## Client-Server vs P2P: 확장성(Scalability) 비교
- 표기법: 서버 업로드 속도 = us, i번째 피어 업로드 = ui, 다운로드 = di, 파일 크기 = F, 피어 수 = N
- 배포 시간(distribution time): 모든 N개 피어에게 파일 사본을 전송하는 데 걸리는 시간
- [Client-Server] Dcs = max(NF/us, F/dmin)
- 서버가 NF 비트를 전부 전송해야 하므로 배포 시간이 N에 비례하여 선형 증가
- [P2P] DP2P = max(F/us, F/dmin, NF/(us + Σui))
- 서버는 파일을 최소 한 번만 전송, 피어들이 재배포하므로 총 업로드 용량이 피어 수에 따라 함께 증가
- 핵심: Client-Server는 N 증가 시 선형으로 무한히 증가 vs P2P는 피어 수가 늘어도 배포 시간이 제한됨 (자체 확장성, self-scalability)
- 피어가 소비자(consumer)를 넘어 재배포자(redistributor) 역할을 수행하기 때문
## BitTorrent
- 가장 대중적인 P2P 파일 배포 프로토콜 (Bram Cohen 개발)
- 토렌트(torrent): 특정 파일 배포에 참여하는 모든 피어들의 모임
- 청크(chunk): 파일을 동일 크기(보통 256KB)로 분할한 단위
- 트래커(tracker): 토렌트 참여 피어들을 추적하는 인프라 노드. 새 피어 참여 시 무작위로 약 50명의 피어 IP 주소 제공
- 이웃 피어(neighboring peers): 트래커에서 받은 피어 목록과 TCP 연결을 수립한 피어들
### 청크 요청 전략: Rarest First
- 이웃 피어들 사이에서 복제본 수가 가장 적은(가장 희귀한) 청크를 우선 요청
- 토렌트 내 각 청크의 복제본 수를 균등하게 만들어 파일 가용성 향상
### 청크 전송 전략: Tit-for-Tat
- Unchoke: 자신에게 가장 높은 속도로 데이터를 공급하는 4명의 피어에게 우선적으로 청크 전송 (매 10초 재계산)
- Optimistic Unchoke: 매 30초마다 무작위로 1명의 추가 피어를 선택하여 청크 전송 → 새로운 거래 파트너 발굴 기회
- Tit-for-Tat(맞대응) 인센티브: 서로 호환되는 업로드 속도의 피어들이 자연스럽게 파트너가 됨 → 프리라이더(freerider) 방지
- DHT(Distributed Hash Table): 레코드가 P2P 시스템의 피어들에 분산 저장되는 분산 데이터베이스
# 2.6 Video Streaming and Content Distribution Networks
## 2.6.1 인터넷 비디오(Internet Video)
- 스트리밍 비디오(Netflix, YouTube, Amazon Prime)는 2020년 인터넷 트래픽의 약 80% 차지
- 비디오: 초당 24 또는 30장의 이미지로 일정한 속도로 재생되는 이미지 연속(sequence of images)
- 압축 방식: 공간적 중복성(spatial, 이미지 내) + 시간적 중복성(temporal, 이미지 간)
- CBR(Constant Bit Rate): 고정 전송률 / VBR(Variable Bit Rate): 가변 전송률
- 비트 전송률: 저품질 100kbps ~ 고화질 4Mbps ~ 4K 10Mbps 이상
- 스트리밍의 핵심 성능 척도: 평균 종단간 처리량(average end-to-end throughput)
- 클라이언트 측 버퍼링(client-side buffering): 네트워크 지연 변동(jitter)을 보상하여 연속 재생 보장
## 2.6.2 HTTP 스트리밍과 DASH
- HTTP 스트리밍: 비디오가 HTTP 서버에 일반 파일로 저장. 클라이언트가 TCP 연결 후 HTTP GET으로 수신
- 단점: 모든 클라이언트가 동일한 인코딩의 비디오를 받음 → 대역폭 차이 무시
- DASH(Dynamic Adaptive Streaming over HTTP): 비디오를 여러 비트 전송률/품질로 인코딩하여 저장
- 클라이언트가 몇 초 길이의 청크(chunk)를 동적으로 요청. 대역폭 높으면 고품질, 낮으면 저품질 선택
- 매니페스트 파일(manifest file): 각 버전의 URL과 비트 전송률 정보 제공
- DASH 클라이언트 지능: (1) 청크 요청 시점 (2) 인코딩 전송률 (3) 청크 요청할 서버 선택
- 핵심: 스트리밍 비디오 = 인코딩(encoding) + DASH + 재생 버퍼링(playout buffering)
## 2.6.3 CDN(Content Distribution Networks)
- 단일 데이터 센터의 문제: (1) 병목 링크 발생 (2) 인기 비디오 중복 전송으로 대역폭 낭비 (3) 단일 장애 지점(single point of failure)
- CDN: 지리적으로 분산된 위치에 서버를 배치하여 콘텐츠 복사본 저장, 사용자 요청을 최적 CDN 위치로 안내
### CDN 배치 철학
- Enter Deep: 전 세계 액세스 ISP 내부로 서버 클러스터 깊이 배치 (Akamai). 사용자와 가까워 지연 감소, 유지보수 복잡
- Bring Home: 소수의 대형 클러스터를 IXP(인터넷 교환소)에 배치 (Limelight). 유지보수 용이, 지연 상대적으로 높을 수 있음
### CDN 동작 메커니즘 (DNS 리디렉션)
- (1) 사용자가 비디오 URL 클릭 → (2) DNS 질의 전송 → (3) 콘텐츠 제공업체 DNS가 CDN 도메인으로 리디렉션 → (4) CDN DNS가 최적 서버 IP 반환 → (5) 클라이언트가 CDN 서버와 TCP 연결 후 HTTP GET
### 클러스터 선택 전략
- 지리적 근접성: Geolocation DB로 LDNS IP를 위치 매핑 → 가장 가까운 클러스터 선택
- 실시간 측정: 클러스터와 클라이언트 간 지연/손실 성능을 주기적으로 프로브로 측정
## 2.6.4 사례 연구: Netflix vs YouTube
### Netflix
- 아마존 클라우드(Amazon Cloud): 웹사이트, 사용자 등록/로그인, 콘텐츠 인제스트, 콘텐츠 프로세싱(다양한 포맷/비트레이트 생성)
- 자체 사설 CDN: IXP(200개 이상) 및 주거용 ISP 내부에 서버 랙 설치
- 푸시 캐싱(push caching): 비수기 시간에 비디오를 CDN 서버에 미리 배포 (풀 캐싱 미사용)
- DNS 리디렉션 대신 아마존 클라우드 소프트웨어가 클라이언트에게 직접 CDN 서버 지정
- DASH 사용: 클라이언트가 약 4초 분량 청크를 동적으로 선택
### YouTube
- Google 자체 사설 CDN 사용: IXP와 ISP에 서버 클러스터 배치
- 풀 캐싱(pull caching) + DNS 리디렉션 사용 (Netflix와 다름)
- HTTP 스트리밍 사용, 사용자가 수동으로 품질 선택 (DASH와 다름)
- 클러스터 선택: RTT가 가장 낮은 클러스터로 안내, 부하 분산을 위해 더 먼 클러스터로 안내하기도 함
# 2.7 소켓 프로그래밍 (Socket Programming)
네트워크 애플리케이션은 클라이언트 프로그램과 서버 프로그램으로 구성되며, 이들은 소켓(socket)을 통해 통신한다. 개발자의 주요 과제는 클라이언트와 서버 코드를 작성하는 것이며, TCP 또는 UDP 중 어떤 프로토콜 위에서 실행할지 결정해야 한다.
- 개방형 애플리케이션 (Open application): RFC 등 표준에 정의된 프로토콜을 구현 → 독립 개발자 간 상호운용 가능 (예: Chrome ↔ Apache)
- 독점 애플리케이션 (Proprietary application): 비공개 프로토콜 사용 → 다른 개발자가 호환 코드 작성 불가
## 2.7.1 UDP 소켓 프로그래밍
UDP는 비연결형(connectionless) 프로토콜로, 송신 프로세스가 패킷을 보내기 전에 목적지 주소(IP 주소 + 포트 번호)를 패킷에 첨부해야 한다.
- 소켓 타입: SOCK_DGRAM (데이터그램 소켓)
- 연결 설정 불필요 — 각 패킷에 목적지 주소(IP + 포트)를 명시적으로 첨부
- 송신자의 소스 주소(IP + 포트)는 OS가 자동으로 첨부
- 클라이언트 소켓 생성 시 포트 번호 지정 불필요 (OS가 자동 할당)
- 서버는 bind()로 특정 포트에 소켓을 바인딩
### UDPClient.py
```python
from socket import *
serverName = 'hostname'
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_DGRAM)
message = input('Input lowercase sentence:')
clientSocket.sendto(message.encode(), (serverName, serverPort))
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode())
clientSocket.close()
```
- socket(AF_INET, SOCK_DGRAM): IPv4 + UDP 소켓 생성
- sendto(): 목적지 주소(serverName, serverPort)를 메시지에 첨부하여 전송
- recvfrom(2048): 서버로부터 수정된 메시지와 서버 주소를 수신 (버퍼 크기 2048)
### UDPServer.py
```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))
print('The server is ready to receive')
while True:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.decode().upper()
    serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```
- bind(('', serverPort)): 포트 12000에 소켓을 바인딩
- recvfrom(): 클라이언트 메시지 + 클라이언트 주소(IP, 포트) 수신
- sendto(): 대문자로 변환한 메시지를 클라이언트 주소로 회신
- while True 루프 → 서버는 무한히 대기하며 여러 클라이언트의 요청을 처리
## 2.7.2 TCP 소켓 프로그래밍
TCP는 연결 지향(connection-oriented) 프로토콜로, 데이터 전송 전에 3-way handshake를 통해 TCP 연결을 설정해야 한다. UDP와 달리 목적지 주소를 명시적으로 첨부할 필요 없이, TCP 연결을 통해 데이터를 전송한다.
- 소켓 타입: SOCK_STREAM (스트림 소켓)
- 연결 설정 필수: connect() → 3-way handshake 수행 (전송 계층에서 수행, 애플리케이션에 투명)
- Welcoming socket (serverSocket): 클라이언트의 초기 연결 요청을 받는 대기용 소켓
- Connection socket (connectionSocket): accept() 호출 시 생성되는 해당 클라이언트 전용 소켓
- TCP는 신뢰성 있는 바이트 스트림 제공 → 순서 보장, 데이터 손실 없음
- send()/recv() 사용 (sendto()/recvfrom() 대신) — 목적지 주소 첨부 불필요
### TCPClient.py
```python
from socket import *
serverName = 'servername'
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverName, serverPort))
sentence = input('Input lowercase sentence:')
clientSocket.send(sentence.encode())
modifiedSentence = clientSocket.recv(1024)
print('From Server: ', modifiedSentence.decode())
clientSocket.close()
```
- socket(AF_INET, SOCK_STREAM): IPv4 + TCP 소켓 생성
- connect((serverName, serverPort)): 서버와 TCP 연결 수립 (3-way handshake)
- send(): 메시지를 TCP 연결에 단순히 넣음 (목적지 주소 첨부 불필요)
- recv(1024): 서버로부터 수정된 메시지 수신
- close(): 소켓 닫음 → TCP에게 FIN 메시지 전송
### TCPServer.py
```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', serverPort))
serverSocket.listen(1)
print('The server is ready to receive')
while True:
    connectionSocket, addr = serverSocket.accept()
    sentence = connectionSocket.recv(1024).decode()
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence.encode())
    connectionSocket.close()
```
- listen(1): TCP 연결 요청 대기 (최대 대기 연결 수 = 1)
- accept(): 클라이언트 연결 수락 → 해당 클라이언트 전용 connectionSocket 생성
- connectionSocket.close(): 해당 클라이언트와의 연결만 종료, serverSocket은 유지되어 다음 클라이언트 수락 가능
## UDP vs TCP 소켓 비교
- 소켓 타입: UDP는 SOCK_DGRAM / TCP는 SOCK_STREAM
- 연결 설정: UDP는 불필요 / TCP는 connect() + 3-way handshake 필수
- 데이터 전송: UDP는 sendto() (목적지 명시) / TCP는 send() (연결 통해 전송)
- 데이터 수신: UDP는 recvfrom() (송신자 주소 함께 반환) / TCP는 recv()
- 서버 소켓: UDP는 소켓 1개로 모든 클라이언트 처리 / TCP는 welcoming socket + 클라이언트별 connection socket
- 신뢰성: UDP는 비신뢰 (순서/전달 보장 없음) / TCP는 신뢰성 있는 순서 보장 전송
# 3.1 전송 계층 소개 및 서비스 (Introduction and Transport-Layer Services)
전송 계층(Transport Layer)은 애플리케이션 계층(Application Layer)과 네트워크 계층(Network Layer) 사이에 위치하며, 서로 다른 호스트에서 실행 중인 애플리케이션 프로세스 간의 논리적 통신(logical communication)을 제공한다.
- 논리적 통신(logical communication): 애플리케이션 관점에서 호스트들이 직접 연결된 것처럼 보이는 것
- 전송 계층 프로토콜은 종단 시스템(end systems)에만 구현되며, 네트워크 라우터에는 구현되지 않음
- 송신 측: 애플리케이션 메시지를 작은 조각으로 분할 → 전송 계층 헤더 추가 → 세그먼트(segment) 생성 → 네트워크 계층으로 전달
- 수신 측: 네트워크 계층이 데이터그램에서 세그먼트 추출 → 전송 계층이 처리하여 수신 애플리케이션에 전달
- 라우터는 데이터그램의 네트워크 계층 필드만 처리하며, 캔슐화된 전송 계층 세그먼트는 검사하지 않음
## 3.1.1 전송 계층과 네트워크 계층의 관계
전송 계층은 프로세스(processes) 간의 논리적 통신을, 네트워크 계층은 호스트(hosts) 간의 논리적 통신을 제공한다. 이 차이를 비유(household analogy)로 설명한다.
### 가정 비유 (Household Analogy)
- 동부 해안과 서부 해안에 각각 12명의 아이가 사는 두 집이 있다
- Ann과 Bill이 각 집에서 우편물 수집/배분 담당 → 사촌들 관점에서 Ann/Bill이 공 우편 서비스 역할
- 대응 관계:
-   • 편지 (letters) = 애플리케이션 메시지 (application messages)
-   • 사촌 (cousins) = 프로세스 (processes)
-   • 집 (houses) = 호스트 (hosts / end systems)
-   • Ann과 Bill = 전송 계층 프로토콜 (transport-layer protocol)
-   • 우편 서비스 (postal service) = 네트워크 계층 프로토콜 (network-layer protocol)
### 핵심
- 전송 프로토콜은 종단 시스템 내에만 존재 — 네트워크 핵심(network core)에서 메시지 이동에는 관여하지 않음
- 전송 프로토콜이 제공할 수 있는 서비스는 기반 네트워크 계층 프로토콜의 서비스 모델에 의해 제한됨
- 단, 네트워크 계층이 제공하지 않는 서비스도 전송 계층에서 제공 가능 (예: 비신뢰적 네트워크 위의 신뢰성 있는 데이터 전송, 암호화 등)
