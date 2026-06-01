# OSI 7계층 정리

## 1. 개요

OSI 7계층은 네트워크 통신 과정을 계층별로 나누어 이해하기 위한 참조 모델이다.

OSI 모델은 통신 과정을 하나의 큰 덩어리로 보지 않고, 역할에 따라 7개의 계층으로 분리한다. 이를 통해 네트워크 구조를 이해하기 쉽고, 장애가 발생했을 때 문제 위치를 계층별로 좁혀갈 수 있다.

OSI 모델은 실제 인터넷 구현 구조 그 자체라기보다는, 네트워크 통신을 이해하고 설명하기 위한 기준 모델로 보는 것이 적절하다.

---

## 2. OSI 7계층이 필요한 이유

네트워크 통신은 사용자가 보기에는 단순해 보인다.

예를 들어 브라우저에서 다음 주소에 접속한다고 가정한다.

```text
https://example.com
```

사용자 입장에서는 “웹사이트에 접속한다”는 하나의 동작이다.

하지만 내부적으로는 다음 과정이 함께 일어난다.

- DNS 질의
- TCP 연결
- TLS 암호화
- HTTP 요청 생성
- IP 주소 기반 라우팅
- MAC 주소 기반 프레임 전달
- 전기 신호 또는 광 신호 전송

이 과정을 계층 없이 한 번에 분석하면 장애 원인을 찾기 어렵다.

OSI 모델은 이 복잡한 통신 과정을 역할별로 분리해서 다음 문제를 해결한다.

| 문제 | OSI 모델의 해결 방식 |
|---|---|
| 통신 과정이 복잡함 | 계층별로 역할을 나누어 이해 |
| 장애 원인 파악이 어려움 | 계층별로 문제 위치 분리 |
| 장비 역할이 혼동됨 | Switch, Router, Firewall 등을 계층 기준으로 구분 |
| 프로토콜 학습이 어려움 | 프로토콜을 계층별로 분류 |
| 벤더별 구현 차이가 있음 | 공통 참조 모델 제공 |

---

## 3. OSI 7계층 구조

| 계층 | 이름 | 데이터 단위(PDU) | 주요 역할 | 대표 장비/프로토콜 |
|---|---|---|---|---|
| 7계층 | Application | Data / Message | 사용자에게 네트워크 서비스 제공 | HTTP, HTTPS, DNS, FTP, SMTP, SSH |
| 6계층 | Presentation | Data | 데이터 표현, 인코딩, 암호화, 압축 | Encoding, Compression, TLS 관련 기능 |
| 5계층 | Session | Data | 통신 세션 연결, 유지, 종료 | Session Control, RPC |
| 4계층 | Transport | Segment / Datagram | 포트 기반 통신, 종단 간 전송 | TCP, UDP |
| 3계층 | Network | Packet | IP 주소 기반 통신, 라우팅 | IP, ICMP, Router, L3 Switch |
| 2계층 | Data Link | Frame | MAC 주소 기반 통신, 같은 네트워크 내 전달 | Ethernet, Switch, VLAN, STP |
| 1계층 | Physical | Bit / Signal | 전기 신호, 광 신호, 케이블, 물리 연결 | Cable, Hub, Repeater |

---

## 4. 계층별 핵심 이해

### 4.1 1계층 Physical Layer

1계층은 케이블, 전기 신호, 광 신호, 포트 연결 상태처럼 물리적인 통신 환경을 담당한다.

```text
Data Unit: Bit / Signal
```

주요 역할:

- 케이블 연결
- 전기 신호 전송
- 광 신호 전송
- 무선 신호 전송
- 포트 Link 상태 처리
- Speed / Duplex 상태 처리

관련 장비:

- Cable
- Hub
- Repeater
- 광 모듈
- 물리 포트

장애 예시:

- 케이블 단선
- 포트 Down
- 장비 전원 문제
- 광 모듈 문제
- Speed / Duplex 불일치
- NIC 불량

확인 명령어:

```bash
show interfaces status
show interfaces
ip link
ethtool eth0
```

---

### 4.2 2계층 Data Link Layer

2계층은 MAC 주소를 기반으로 같은 네트워크 안에서 프레임을 전달하는 계층이다.

```text
Data Unit: Frame
Address: MAC Address
```

주요 역할:

- MAC 주소 기반 통신
- Ethernet Frame 전달
- 같은 VLAN 내 통신
- VLAN 처리
- Access / Trunk Port 처리
- STP를 통한 Loop 방지
- Frame 오류 검출

관련 장비/기술:

- Switch
- Bridge
- Ethernet
- VLAN
- Trunk
- STP

장애 예시:

- VLAN 할당 오류
- Access / Trunk 설정 오류
- MAC Address Table 학습 실패
- STP Blocking 상태
- Native VLAN 불일치
- Duplex mismatch

확인 명령어:

```bash
show mac address-table
show vlan brief
show interfaces trunk
show spanning-tree
ip link
bridge link
bridge vlan show
```

---

### 4.3 3계층 Network Layer

3계층은 IP 주소를 기반으로 서로 다른 네트워크 간 통신을 담당한다.

```text
Data Unit: Packet
Address: IP Address
```

주요 역할:

- IP 주소 기반 통신
- Routing
- Packet 전달
- Subnet 구분
- 목적지 네트워크까지 경로 결정
- TTL 처리
- ICMP 기반 상태 확인

관련 장비/프로토콜:

- Router
- L3 Switch
- IPv4
- IPv6
- ICMP
- OSPF
- BGP
- Static Routing

장애 예시:

- IP 주소 오설정
- Subnet Mask 오류
- Default Gateway 누락
- Routing Table 누락
- ACL 차단
- ICMP 차단
- NAT 설정 오류

확인 명령어:

```bash
show ip interface brief
show ip route
ping
traceroute
ip addr
ip route
```

---

### 4.4 4계층 Transport Layer

4계층은 종단 간 통신을 담당한다.

```text
TCP: Segment
UDP: Datagram
Address: Port Number
```

주요 역할:

- TCP/UDP 포트 기반 통신
- 애플리케이션 구분
- TCP 연결 관리
- TCP 신뢰성 보장
- TCP 흐름 제어
- TCP 재전송
- UDP 비연결형 전송

예시:

```text
HTTP  : TCP 80
HTTPS : TCP 443
SSH   : TCP 22
DNS   : UDP/TCP 53
```

장애 예시:

- TCP 3-way Handshake 실패
- Port 미개방
- 방화벽 포트 차단
- Connection Timeout
- Connection Refused
- 서비스는 실행 중이지만 특정 포트가 Listen하지 않음

확인 명령어:

```bash
ss -tulnp
netstat -tulnp
nc -vz [IP] [PORT]
telnet [IP] [PORT]
netstat -ano
Test-NetConnection [IP] -Port [PORT]
```

---

### 4.5 5계층 Session Layer

5계층은 통신 세션을 생성, 유지, 종료하는 역할을 한다.

```text
Data Unit: Data
```

주요 역할:

- 세션 생성
- 세션 유지
- 세션 종료
- 연결 상태 관리
- 재연결 또는 체크포인트 개념

관련 예시:

- 로그인 세션
- 원격 접속 세션
- RPC
- SMB Session
- 애플리케이션 세션 유지

장애 예시:

- 로그인 세션 만료
- 세션 타임아웃
- 연결 유지 실패
- 인증 후 세션 저장 실패

---

### 4.6 6계층 Presentation Layer

6계층은 데이터 표현 방식을 담당한다.

```text
Data Unit: Data
```

주요 역할:

- Encoding
- Decoding
- Encryption
- Decryption
- Compression
- 데이터 형식 변환

관련 예시:

- 문자 인코딩
- 압축
- 직렬화 형식
- TLS/SSL 관련 암호화 기능
- 인증서 검증

주의:

TLS/SSL은 OSI 모델 설명에서 Presentation Layer 예시로 자주 언급되지만, 실제 TCP/IP 구현에서는 OSI 6계층에 딱 맞게 분리되어 동작한다고 단정하기 어렵다. 실무에서는 Application 계층과 Transport 계층 사이에서 동작하는 보안 계층으로 이해하는 것이 더 현실적이다.

확인 명령어:

```bash
openssl s_client -connect [DOMAIN]:443
curl -v https://[DOMAIN]
```

---

### 4.7 7계층 Application Layer

7계층은 사용자가 직접 사용하는 애플리케이션 서비스와 가장 가까운 계층이다.

```text
Data Unit: Data / Message
```

주요 역할:

- 사용자 서비스 제공
- 애플리케이션 프로토콜 처리
- 요청/응답 처리
- 인증/권한 처리
- 애플리케이션 로그 생성

관련 프로토콜:

- HTTP
- HTTPS
- DNS
- FTP
- SMTP
- SSH
- DHCP
- SNMP

장애 예시:

- DNS 질의 실패
- 웹 서버 4xx/5xx 응답
- 서비스 프로세스 중지
- 애플리케이션 설정 오류
- 인증 실패
- 권한 문제

확인 명령어:

```bash
nslookup [DOMAIN]
dig [DOMAIN]
curl -I http://[IP]
curl -I https://[DOMAIN]
systemctl status [service]
journalctl -u [service]
```

---

## 5. 데이터 단위 정리

| 계층 | PDU |
|---|---|
| 7계층 Application | Data / Message |
| 6계층 Presentation | Data |
| 5계층 Session | Data |
| 4계층 Transport | Segment / Datagram |
| 3계층 Network | Packet |
| 2계층 Data Link | Frame |
| 1계층 Physical | Bit / Signal |

---

## 6. Segment / Packet / Frame 구분

Segment, Packet, Frame은 서로 다른 계층에서 바라본 데이터의 이름이다.

### Segment

4계층 Transport Layer의 데이터 단위이다.

```text
[TCP Header][Application Data]
```

### Packet

3계층 Network Layer의 데이터 단위이다.

```text
[IP Header][TCP Segment]
```

### Frame

2계층 Data Link Layer의 데이터 단위이다.

```text
[Ethernet Header][IP Packet][Ethernet Trailer]
```

관계 정리:

```text
Frame
└── Packet
    └── Segment
        └── Data
```

---

## 7. Encapsulation / Decapsulation

### Encapsulation

송신 측에서 데이터가 하위 계층으로 내려가면서 각 계층의 Header 또는 Trailer가 추가되는 과정이다.

```text
Application Data
↓
TCP Header 추가
↓
IP Header 추가
↓
Ethernet Header / Trailer 추가
↓
Bit 또는 Signal로 변환
```

계층별 표현:

```text
L7 Data
↓
L4 Segment
↓
L3 Packet
↓
L2 Frame
↓
L1 Bit
```

구조:

```text
[Ethernet Header]
    [IP Header]
        [TCP Header]
            [Application Data]
[Ethernet Trailer]
```

### Decapsulation

수신 측에서 데이터를 상위 계층으로 올리면서 각 계층의 Header 또는 Trailer를 제거하고 해석하는 과정이다.

```text
Bit
↓
Frame 확인
↓
Packet 확인
↓
Segment 확인
↓
Application Data 전달
```

---

## 8. 실제 통신 흐름 예시

사용자 PC가 웹 서버에 접속한다고 가정한다.

```text
사용자 PC → 스위치 → 라우터 → 인터넷 → 라우터 → 스위치 → 웹 서버
```

계층별 흐름:

| 단계 | 관련 계층 | 설명 |
|---|---|---|
| 브라우저에서 HTTP/HTTPS 요청 생성 | 7계층 | 애플리케이션 요청 생성 |
| TLS 암호화 처리 | 6계층 관점 | HTTPS 암호화 |
| TCP 연결 생성 | 4계층 | Port 기반 연결 |
| IP Packet 생성 | 3계층 | 출발지/목적지 IP 포함 |
| Ethernet Frame 생성 | 2계층 | 출발지/목적지 MAC 포함 |
| 신호 전송 | 1계층 | 전기/광/무선 신호 전송 |

---

## 9. 네트워크 장비와 OSI 계층

| 장비 | 주 관련 계층 | 기준 정보 | 설명 |
|---|---|---|---|
| Hub | L1 | Bit / Signal | 들어온 신호를 다른 포트로 반복 전달 |
| Switch | L2 | MAC Address | MAC Address Table 기반 Frame 전달 |
| L3 Switch | L3 | IP Address | 스위칭 기능과 라우팅 기능을 함께 제공 |
| Router | L3 | IP Address | Routing Table 기반 Packet 전달 |
| Firewall | L3~L4, 일부 L7 | IP, Port, Protocol, Application | 정책 기반 허용/차단 |
| Load Balancer | L4 또는 L7 | IP/Port 또는 HTTP 정보 | 서버 부하 분산 |

---

## 10. OSI 계층별 장애 점검 흐름

| 순서 | 확인 항목 | 관련 계층 |
|---|---|---|
| 1 | 케이블, 포트 Link, 장비 전원 확인 | 1계층 |
| 2 | VLAN, MAC Table, Trunk, STP 확인 | 2계층 |
| 3 | IP 주소, Subnet, Gateway 확인 | 3계층 |
| 4 | Routing Table, Ping, Traceroute 확인 | 3계층 |
| 5 | TCP/UDP 포트 Open 여부 확인 | 4계층 |
| 6 | TLS 인증서, 암호화 문제 확인 | 6계층 |
| 7 | DNS, 웹 서비스, 애플리케이션 로그 확인 | 7계층 |

---

## 11. 예시 상황: 사용자 PC에서 서버 접속 실패

```text
1. PC의 Link 상태 확인
2. Switch Port Up 여부 확인
3. VLAN 할당 확인
4. PC IP/Subnet/Gateway 확인
5. Gateway Ping 확인
6. 목적지 서버 Ping 확인
7. Traceroute로 경로 확인
8. TCP 80/443 Port 확인
9. DNS 질의 확인
10. curl로 HTTP/HTTPS 응답 확인
11. 서버 애플리케이션 로그 확인
```

---

## 12. TCP/IP Model과 OSI Model 비교

| OSI 7계층 | TCP/IP 모델 |
|---|---|
| 7계층 Application | Application |
| 6계층 Presentation | Application |
| 5계층 Session | Application |
| 4계층 Transport | Transport |
| 3계층 Network | Internet |
| 2계층 Data Link | Link / Network Access |
| 1계층 Physical | Link / Network Access |

### OSI 모델

- 참조 모델 성격이 강하다.
- 계층 구분이 세밀하다.
- 네트워크 학습과 장애 분석에 유용하다.
- 실제 구현과 1:1로 완전히 일치하지는 않는다.

### TCP/IP 모델

- 실제 인터넷 구현과 더 가깝다.
- Application, Transport, Internet, Link 계층 중심으로 설명한다.
- TCP, UDP, IP, ICMP, Ethernet 같은 실제 프로토콜과 연결된다.

---

## 13. 실무 관점 정리

OSI 7계층은 단순 암기용 개념이 아니라 장애 원인을 계층별로 나누어 확인하기 위한 기준이다.

자주 확인하는 계층:

- 1계층: 케이블, 포트, 인터페이스 상태
- 2계층: MAC Address, VLAN, Trunk, STP
- 3계층: IP, Subnet, Gateway, Routing
- 4계층: TCP/UDP Port
- 7계층: DNS, HTTP/HTTPS, 서비스 상태

실무에서 5계층과 6계층은 독립적으로 분리해서 점검하기보다는, 애플리케이션 연결 상태, TLS 인증서, 암호화, 세션 유지 문제와 함께 확인하는 경우가 많다.

---

## 14. 핵심 요약

- OSI 7계층은 네트워크 통신을 역할별로 나누어 이해하기 위한 참조 모델이다.
- 송신 측에서는 Encapsulation이 발생한다.
- 수신 측에서는 Decapsulation이 발생한다.
- L4 데이터 단위는 Segment 또는 Datagram이다.
- L3 데이터 단위는 Packet이다.
- L2 데이터 단위는 Frame이다.
- Switch는 주로 MAC 주소를 기준으로 동작한다.
- Router는 IP 주소를 기준으로 동작한다.
- Firewall은 IP, Port, Protocol, Application 정보를 기준으로 정책을 적용할 수 있다.
- Load Balancer는 L4 또는 L7 기준으로 트래픽을 분산할 수 있다.
- 장애 분석은 OSI 계층 기준으로 아래에서 위로 확인하면 원인을 좁히기 쉽다.

---

## 15. 다음 학습 연결

OSI 7계층을 이해한 뒤에는 데이터가 각 계층을 지나며 Header와 Trailer가 붙는 과정을 더 자세히 이해해야 한다.

다음 학습 주제:

- TCP/IP Model
- Encapsulation / Decapsulation
- Packet / Frame / Segment 구조
- Ethernet Frame 구조
- ARP
- IP Header
- TCP / UDP Header
- Wireshark 패킷 분석
