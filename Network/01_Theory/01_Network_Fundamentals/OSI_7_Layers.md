# OSI 7계층 정리

## 1. 개요

OSI 7계층은 네트워크 통신 과정을 계층별로 나누어 이해하기 위한 모델이다.  
실무에서는 단순 암기보다 장애 원인을 계층별로 분리해서 확인하는 기준으로 활용된다.

예를 들어 통신이 되지 않을 때 물리 연결 문제인지, VLAN 문제인지, IP 설정 문제인지, 라우팅 문제인지, 방화벽 정책 문제인지 구분하기 위해 OSI 계층 관점으로 접근할 수 있다.

---

## 2. OSI 7계층 구조

| 계층 | 이름 | 주요 역할 | 대표 장비/프로토콜 |
|---|---|---|---|
| 7계층 | Application | 사용자에게 네트워크 서비스를 제공 | HTTP, HTTPS, DNS, FTP, SMTP, SSH |
| 6계층 | Presentation | 데이터 표현, 인코딩, 암호화, 압축 | TLS, SSL, Encoding |
| 5계층 | Session | 통신 세션 연결, 유지, 종료 | Session Control |
| 4계층 | Transport | 포트 기반 통신, 신뢰성 제어 | TCP, UDP |
| 3계층 | Network | IP 주소 기반 통신, 라우팅 | IP, ICMP, Router, L3 Switch |
| 2계층 | Data Link | MAC 주소 기반 통신, 같은 네트워크 내 프레임 전달 | Ethernet, Switch, VLAN, STP |
| 1계층 | Physical | 전기적 신호, 케이블, 물리 연결 | Cable, Hub, NIC |

---

## 3. 계층별 핵심 이해

### 1계층 Physical Layer

1계층은 케이블, 전기 신호, 포트 연결 상태처럼 물리적인 통신 환경을 담당한다.

장애 확인 시에는 먼저 케이블 연결, 장비 전원, 포트 Link 상태, 인터페이스 Up/Down 여부를 확인한다.

확인 예시:

```bash
show interfaces status
show interfaces
```

주요 확인 항목:

- 케이블 연결 상태
- 포트 Link 상태
- 장비 전원 상태
- 인터페이스 물리 장애 여부

---

### 2계층 Data Link Layer

2계층은 MAC 주소를 기반으로 같은 네트워크 안에서 프레임을 전달하는 계층이다.  
스위치, VLAN, Trunk, STP가 이 계층과 관련된다.

스위치는 MAC Address Table을 이용해 어느 포트에 어떤 MAC 주소가 있는지 학습하고, 목적지 MAC 주소에 따라 프레임을 전달한다.

주요 확인 항목:

- MAC Address Table
- VLAN 할당 상태
- Access Port / Trunk Port 설정
- STP 상태
- 같은 VLAN 내 통신 가능 여부

확인 예시:

```bash
show mac address-table
show vlan brief
show interfaces trunk
show spanning-tree
```

---

### 3계층 Network Layer

3계층은 IP 주소를 기반으로 서로 다른 네트워크 간 통신을 담당한다.  
라우터 또는 L3 스위치가 이 계층에서 동작하며, Routing Table을 기준으로 목적지 네트워크까지의 경로를 결정한다.

주요 확인 항목:

- IP 주소
- Subnet Mask
- Default Gateway
- Routing Table
- ICMP 응답 여부
- 목적지 네트워크까지의 경로

확인 예시:

```bash
ping
traceroute
show ip route
show ip interface brief
```

장애 예시:

- IP 주소 오설정
- Subnet Mask 오류
- Gateway 미설정
- Routing Table 누락
- ACL 또는 방화벽 정책 차단

---

### 4계층 Transport Layer

4계층은 TCP/UDP 포트를 기반으로 통신을 구분한다.  
TCP는 연결성과 신뢰성을 제공하고, UDP는 빠른 전송에 초점을 둔다.

예를 들어 웹 서비스는 보통 TCP 80 또는 TCP 443 포트를 사용하고, DNS는 UDP 53 또는 TCP 53을 사용할 수 있다.

주요 확인 항목:

- TCP/UDP 사용 여부
- 서비스 포트 Open 여부
- 세션 연결 상태
- 방화벽 포트 허용 여부

확인 예시:

```bash
netstat -ano
ss -tulnp
telnet [IP] [PORT]
nc -vz [IP] [PORT]
```

---

### 5계층 Session Layer

5계층은 통신 세션을 생성, 유지, 종료하는 역할을 한다.  
실무에서 독립적으로 분리해서 점검하기보다는, 인증 세션이나 연결 유지 상태를 확인할 때 함께 고려한다.

예를 들어 웹 로그인 세션, 원격 접속 세션, 애플리케이션 세션 유지 문제가 이 계층과 관련될 수 있다.

주요 확인 항목:

- 세션 유지 여부
- 로그인 세션 만료 여부
- 연결이 중간에 끊기는지 여부

---

### 6계층 Presentation Layer

6계층은 데이터 표현 방식, 인코딩, 암호화, 압축과 관련된다.  
대표적으로 TLS/SSL 암호화, 문자 인코딩, 인증서 문제가 이 계층과 관련될 수 있다.

예를 들어 HTTPS 접속 시 인증서 오류가 발생하거나, 브라우저에서 TLS 관련 오류가 발생하는 경우 6계층 관점에서 확인할 수 있다.

주요 확인 항목:

- TLS/SSL 인증서 상태
- 암호화 프로토콜 지원 여부
- 인코딩 문제
- 인증서 신뢰 여부

확인 예시:

```bash
openssl s_client -connect [DOMAIN]:443
curl -v https://[DOMAIN]
```

---

### 7계층 Application Layer

7계층은 사용자가 직접 사용하는 애플리케이션 서비스와 가장 가까운 계층이다.  
HTTP, HTTPS, DNS, FTP, SMTP, SSH 같은 서비스가 이 계층과 관련된다.

네트워크 연결이 정상이어도 서비스 자체가 중지되어 있거나, 애플리케이션 설정이 잘못되어 있으면 접속이 실패할 수 있다.

주요 확인 항목:

- 서비스 실행 상태
- DNS 질의 결과
- 웹 서버 응답 상태
- 애플리케이션 로그
- 인증 및 권한 설정

확인 예시:

```bash
nslookup [DOMAIN]
curl -I http://[IP]
curl -I https://[DOMAIN]
systemctl status [service]
journalctl -u [service]
```

---

## 4. 계층별 장애 점검 흐름

네트워크 장애를 확인할 때는 아래 순서로 접근하면 원인을 좁히기 쉽다.

| 순서 | 확인 항목 | 관련 계층 |
|---|---|---|
| 1 | 케이블, 포트 Link, 장비 전원 확인 | 1계층 |
| 2 | VLAN, MAC Table, Trunk, STP 확인 | 2계층 |
| 3 | IP 주소, Subnet, Gateway 확인 | 3계층 |
| 4 | Routing Table, Ping, Traceroute 확인 | 3계층 |
| 5 | TCP/UDP 포트 Open 여부 확인 | 4계층 |
| 6 | TLS/SSL 인증서, 암호화 문제 확인 | 6계층 |
| 7 | DNS, 웹 서비스, 애플리케이션 로그 확인 | 7계층 |

---

## 5. 예시 상황

### 상황

사용자 PC에서 서버 접속이 되지 않는다.

### 계층별 확인 흐름

| 확인 단계 | 관련 계층 | 확인 내용 |
|---|---|---|
| 물리 연결 확인 | 1계층 | 케이블 연결, 포트 Link 상태 |
| 스위치 설정 확인 | 2계층 | VLAN 할당, Trunk 설정, MAC Address Table |
| IP 설정 확인 | 3계층 | IP 주소, Subnet Mask, Default Gateway |
| 경로 확인 | 3계층 | Routing Table, Ping, Traceroute |
| 포트 확인 | 4계층 | TCP/UDP 포트 접근 가능 여부 |
| 암호화 확인 | 6계층 | TLS/SSL 인증서 오류 여부 |
| 서비스 확인 | 7계층 | 웹 서버, DNS, SSH 등 서비스 동작 여부 |

---

## 6. 실무 관점 정리

OSI 7계층은 단순 암기용 개념이 아니라 장애 원인을 계층별로 나누어 확인하기 위한 기준이다.

네트워크 엔지니어 관점에서는 특히 아래 계층을 자주 확인하게 된다.

- 1계층: 케이블, 포트, 인터페이스 상태
- 2계층: MAC Address, VLAN, Trunk, STP
- 3계층: IP, Subnet, Gateway, Routing
- 4계층: TCP/UDP Port
- 7계층: DNS, HTTP/HTTPS, 서비스 상태

따라서 이후 학습할 VLAN, STP, Routing, NAT, ACL, VPN, DNS, DHCP, ARP, TCP/UDP도 OSI 계층 구조와 연결해서 정리할 필요가 있다.

---

## 7. 다음 학습 연결

OSI 7계층을 이해한 뒤에는 데이터가 각 계층을 지나며 Header와 Trailer가 붙는 과정을 이해해야 한다.

다음 학습 주제:

- Encapsulation / Decapsulation
- Ethernet Frame 구조
- IP Header
- TCP / UDP Header
- Wireshark 패킷 분석
