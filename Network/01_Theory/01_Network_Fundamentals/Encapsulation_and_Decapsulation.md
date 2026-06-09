# Encapsulation and Decapsulation

> 경로: `Network/01_Theory/01_Network_Fundamentals/Encapsulation_and_Decapsulation.md`  
> 연결 문서: [`OSI_7_Layers.md`](./OSI_7_Layers.md), [`TCP_IP_Model.md`](./TCP_IP_Model.md)  
> 학습 흐름: `OSI 7계층` → `TCP/IP Model` → `Encapsulation / Decapsulation` → `IP Address` → `ARP` → `ICMP` → `Packet Analysis`

---

## 1. 학습 목표

이 문서의 목표는 네트워크 통신에서 **Encapsulation**과 **Decapsulation**이 왜 필요한지 이해하고, 실제 패킷 분석과 장애 분석에 연결할 수 있게 정리하는 것이다.

이 문서를 통해 다음을 이해한다.

- Encapsulation이 왜 필요한지 이해한다.
- Decapsulation이 왜 필요한지 이해한다.
- 송신자가 데이터를 보낼 때 계층별로 데이터가 어떻게 감싸지는지 이해한다.
- 수신자가 데이터를 받을 때 Frame에서 시작해 상위 계층으로 어떻게 해석되는지 이해한다.
- Header, Payload, Trailer의 개념을 구분한다.
- Segment, Packet, Frame을 OSI / TCP-IP 계층 관점에서 구분한다.
- 스위치, 라우터, 방화벽이 Encapsulation된 데이터의 어느 부분을 보고 처리하는지 이해한다.
- Wireshark에서 Encapsulation 구조를 확인하는 방법을 이해한다.
- 장애 분석에서 Encapsulation / Decapsulation 개념이 왜 중요한지 이해한다.

---

## 2. 핵심 요약

Encapsulation은 송신자가 상위 계층 데이터를 하위 계층으로 내려보내면서 각 계층의 Header 또는 Trailer를 붙이는 과정이다.

Decapsulation은 수신자가 하위 계층에서 받은 데이터를 상위 계층으로 올리면서 각 계층의 Header 또는 Trailer를 해석하고 제거하는 과정이다.

```text
송신 측 Encapsulation

Application Data
      ↓
TCP Segment / UDP Datagram
      ↓
IP Packet / IP Datagram
      ↓
Ethernet Frame / Wi-Fi Frame
      ↓
Bits / Signal
```

```text
수신 측 Decapsulation

Bits / Signal
      ↓
Ethernet Frame / Wi-Fi Frame
      ↓
IP Packet / IP Datagram
      ↓
TCP Segment / UDP Datagram
      ↓
Application Data
```

각 계층은 자기 역할에 필요한 정보를 Header 또는 Trailer에 담는다.

| 계층 | 주로 추가하는 정보 | 데이터 단위 |
|---|---|---|
| Application | 애플리케이션 요청/응답 정보 | Data, Message |
| Transport | 포트 번호, TCP 제어 정보, UDP 정보 | Segment, Datagram |
| Internet | 출발지 IP, 목적지 IP, TTL/Hop Limit | Packet, Datagram |
| Network Access | 출발지 MAC, 목적지 MAC, FCS 등 | Frame |

---

## 3. Encapsulation이 필요한 이유

Encapsulation이 필요한 이유는 **애플리케이션 데이터만으로는 네트워크를 통해 목적지까지 전달될 수 없기 때문**이다.

예를 들어 브라우저가 다음과 같은 HTTP 요청을 만들었다고 하자.

```text
GET / HTTP/1.1
Host: www.example.com
```

이 데이터만으로는 다음 질문에 답할 수 없다.

```text
어떤 서버 IP로 보낼 것인가?
어떤 애플리케이션 포트로 보낼 것인가?
다음 홉 장비의 MAC 주소는 무엇인가?
패킷이 네트워크를 지나며 얼마나 오래 살아 있을 수 있는가?
프레임이 전송 중 손상되었는지 어떻게 확인할 것인가?
```

이 질문에 답하기 위해 각 계층은 자기 역할에 맞는 정보를 붙인다.

---

### 3.1 계층별 역할을 분리하기 위해

Encapsulation은 통신 기능을 계층별로 분리한다.

```text
Application Layer: 무엇을 보낼 것인가?
Transport Layer: 어떤 프로세스끼리 통신할 것인가?
Internet Layer: 어떤 IP 주소로 보낼 것인가?
Network Access Layer: 다음 홉까지 어떤 프레임으로 보낼 것인가?
```

이렇게 역할을 나누면 하나의 계층이 바뀌어도 다른 계층을 그대로 유지할 수 있다.

예를 들어 HTTP 데이터는 TCP 위에서도 전달될 수 있고, HTTP/3처럼 QUIC 위에서도 전달될 수 있다.  
또 IP 패킷은 Ethernet, Wi-Fi 등 다양한 링크 기술 위에서 전달될 수 있다.

---

### 3.2 주소 정보를 단계별로 추가하기 위해

네트워크 통신에는 여러 종류의 주소가 사용된다.

| 주소 종류 | 사용 계층 | 목적 |
|---|---|---|
| URL / Domain Name | Application | 사용자가 접근할 서비스 식별 |
| Port Number | Transport | 호스트 내부의 애플리케이션 프로세스 식별 |
| IP Address | Internet | 네트워크 간 목적지 호스트 식별 |
| MAC Address | Network Access | 같은 링크에서 다음 홉 장비 식별 |

Encapsulation 과정에서 이 주소 정보들이 계층별 Header에 들어간다.

```text
HTTP Data
  + TCP Header: Source Port, Destination Port
  + IP Header: Source IP, Destination IP
  + Ethernet Header: Source MAC, Destination MAC
```

---

### 3.3 장비가 필요한 정보만 보고 처리할 수 있게 하기 위해

모든 네트워크 장비가 애플리케이션 데이터를 전부 볼 필요는 없다.

| 장비 | 주로 확인하는 정보 |
|---|---|
| 스위치 | Ethernet Header의 MAC 주소 |
| 라우터 | IP Header의 목적지 IP 주소 |
| 방화벽 | IP, Port, Protocol, Session, Application 정보 |
| 서버 애플리케이션 | HTTP, DNS, SSH 같은 Application 데이터 |

Encapsulation 덕분에 장비는 자신의 역할에 필요한 계층의 Header를 보고 처리할 수 있다.

---

### 3.4 장애 분석 기준을 만들기 위해

Encapsulation 구조를 알면 장애를 계층별로 분리해서 볼 수 있다.

```text
Frame이 안 보인다       → Network Access 문제 가능성
ARP가 안 된다           → 같은 링크의 주소 해석 문제 가능성
IP 응답이 없다          → 라우팅, 방화벽, 경로 문제 가능성
TCP SYN-ACK이 없다      → 포트 차단, 서버 미동작, 방화벽 문제 가능성
HTTP 응답이 이상하다    → 애플리케이션, 인증, 프록시, TLS 문제 가능성
```

즉, Encapsulation은 단순한 이론이 아니라 Wireshark, tcpdump, 방화벽 로그, 서버 로그를 해석하는 기본 구조이다.

---

## 4. Decapsulation이 필요한 이유

Decapsulation은 수신자가 받은 데이터를 계층별로 해석하고 최종 애플리케이션에 전달하기 위해 필요하다.

송신자가 데이터를 보낼 때 여러 계층의 Header와 Trailer가 붙었다면, 수신자는 반대로 다음 순서로 처리한다.

```text
Ethernet Frame 수신
    ↓
Ethernet Header / Trailer 확인
    ↓
IP Packet 추출
    ↓
IP Header 확인
    ↓
TCP Segment 또는 UDP Datagram 추출
    ↓
Transport Header 확인
    ↓
Application Data 추출
    ↓
애플리케이션에 전달
```

수신자는 각 계층에서 다음을 확인한다.

| 계층 | Decapsulation 시 확인하는 것 |
|---|---|
| Network Access | 이 Frame이 나에게 온 것인지, 손상되었는지 |
| Internet | 목적지 IP가 나인지, 상위 프로토콜이 무엇인지 |
| Transport | 목적지 포트가 열려 있는지, 어떤 프로세스에 전달할지 |
| Application | 요청 형식, 인증, 세션, 데이터 내용을 어떻게 처리할지 |

---

## 5. Header, Payload, Trailer

Encapsulation을 이해하려면 Header, Payload, Trailer를 구분해야 한다.

---

### 5.1 Header

Header는 각 계층이 데이터를 처리하기 위해 앞쪽에 붙이는 제어 정보이다.

예를 들어 TCP Header에는 다음과 같은 정보가 들어간다.

```text
Source Port
Destination Port
Sequence Number
Acknowledgment Number
Flags
Window Size
Checksum
```

IP Header에는 다음과 같은 정보가 들어간다.

```text
Source IP Address
Destination IP Address
TTL 또는 Hop Limit
Protocol 또는 Next Header
Fragmentation 관련 정보
```

Ethernet Header에는 일반적으로 다음과 같은 정보가 들어간다.

```text
Destination MAC Address
Source MAC Address
EtherType 또는 Length
VLAN Tag가 있는 경우 802.1Q 관련 정보
```

---

### 5.2 Payload

Payload는 해당 계층이 실제로 운반하는 상위 계층 데이터이다.

예를 들어 TCP 관점에서 Payload는 애플리케이션 데이터이다.

```text
[TCP Header][TCP Payload]
```

IP 관점에서 Payload는 TCP Segment 또는 UDP Datagram이다.

```text
[IP Header][IP Payload: TCP Segment or UDP Datagram]
```

Ethernet 관점에서 Payload는 보통 IP Packet, ARP Message 등이다.

```text
[Ethernet Header][Ethernet Payload: IP Packet or ARP Message][Ethernet Trailer]
```

중요한 점은 **한 계층의 전체 데이터 단위가 아래 계층에서는 Payload가 된다**는 것이다.

```text
TCP Segment는 IP 계층의 Payload가 된다.
IP Packet은 Ethernet 계층의 Payload가 된다.
```

---

### 5.3 Trailer

Trailer는 데이터 뒤쪽에 붙는 제어 정보이다.

모든 프로토콜이 Trailer를 사용하는 것은 아니다.  
Ethernet Frame의 FCS(Frame Check Sequence)는 대표적인 Trailer 예시이다.

```text
[Ethernet Header][Payload][FCS]
```

FCS는 프레임 전송 중 오류가 발생했는지 확인하는 데 사용된다.  
다만 일반적인 PC 환경에서 Wireshark 캡처를 보면 Ethernet FCS가 보이지 않는 경우가 많다. NIC나 드라이버가 FCS를 운영체제에 전달하지 않을 수 있기 때문이다.

---

## 6. 송신 측 Encapsulation 흐름

예시 상황은 다음과 같다.

```text
사용자 PC → https://www.example.com 접속
```

여기서는 TCP 기반 HTTPS, 즉 HTTP/1.1 또는 HTTP/2 over TLS의 일반적인 흐름을 기준으로 설명한다.  
HTTP/3은 TCP가 아니라 QUIC을 사용하며, QUIC은 UDP 기반이다.

---

### 6.1 Application Layer

브라우저는 사용자의 요청을 애플리케이션 데이터로 만든다.

```text
GET / HTTP/1.1
Host: www.example.com
```

HTTPS에서는 HTTP 데이터가 TLS를 통해 보호된다.

```text
[HTTP Data]
    ↓
[TLS Protected Application Data]
```

Application 계층의 관심사는 다음과 같다.

```text
어떤 URL에 접근하는가?
어떤 Host를 요청하는가?
어떤 HTTP Method를 사용하는가?
인증 정보나 Cookie가 있는가?
TLS 인증서 검증이 필요한가?
```

Application 계층의 데이터 단위는 일반적으로 `Data` 또는 `Message`라고 부른다.

---

### 6.2 Transport Layer

Transport 계층은 애플리케이션 데이터를 TCP Segment 또는 UDP Datagram으로 만든다.

TCP 기반 HTTPS 예시:

```text
[TCP Header][TLS Protected HTTP Data]
```

TCP Header에는 대표적으로 다음 정보가 들어간다.

```text
Source Port
Destination Port
Sequence Number
Acknowledgment Number
Flags
Window Size
Checksum
```

예를 들어 클라이언트가 서버의 HTTPS 서비스에 접속한다면 다음처럼 볼 수 있다.

```text
Source Port: 51524
Destination Port: 443
```

여기서 Source Port는 클라이언트가 임시로 사용하는 Ephemeral Port이고, Destination Port는 서버의 HTTPS 포트이다.

TCP는 애플리케이션에 신뢰성 있는 순서화된 바이트 스트림을 제공한다.  
애플리케이션의 바이트 스트림은 TCP Segment로 나뉘고, 각 TCP Segment는 IP Datagram에 실려 전달된다.

UDP 기반 통신의 경우에는 다음과 같이 UDP Header가 붙는다.

```text
[UDP Header][Application Data]
```

UDP Header에는 대표적으로 다음 정보가 들어간다.

```text
Source Port
Destination Port
Length
Checksum
```

UDP 자체는 연결 설정, 재전송, 순서 보장, 흐름 제어를 제공하지 않는다.  
다만 QUIC처럼 UDP 위에서 별도의 연결 관리, 신뢰성, 암호화 기능을 구현하는 프로토콜도 있다.

---

### 6.3 Internet Layer

Internet 계층은 TCP Segment 또는 UDP Datagram에 IP Header를 붙여 IP Packet 또는 IP Datagram을 만든다.

IPv4 예시:

```text
[IPv4 Header][TCP Segment]
```

IPv6 예시:

```text
[IPv6 Header][TCP Segment]
```

IP Header에는 대표적으로 다음 정보가 들어간다.

```text
Source IP Address
Destination IP Address
TTL 또는 Hop Limit
Protocol 또는 Next Header
Fragmentation 관련 정보
```

Internet 계층의 핵심 질문은 다음과 같다.

```text
목적지 IP가 어디인가?
목적지가 같은 서브넷에 있는가?
아니면 기본 게이트웨이로 보내야 하는가?
다음 홉은 어디인가?
```

예를 들어 목적지 서버가 다른 네트워크에 있다면, IP Packet의 목적지 IP는 웹 서버 IP로 설정되지만, 실제 Ethernet Frame의 목적지 MAC 주소는 기본 게이트웨이의 MAC 주소가 된다.

```text
Destination IP  = 최종 목적지 서버
Destination MAC = 현재 링크의 다음 홉, 예: 기본 게이트웨이
```

---

### 6.4 Network Access Layer

Network Access 계층은 IP Packet을 Frame 안에 넣는다.

Ethernet 예시:

```text
[Ethernet Header][IP Packet][Ethernet Trailer]
```

Ethernet Header에는 일반적으로 다음 정보가 들어간다.

```text
Destination MAC Address
Source MAC Address
EtherType
VLAN Tag가 있는 경우 802.1Q 정보
```

Ethernet Trailer에는 FCS가 포함될 수 있다.

```text
Frame Check Sequence
```

최종적으로 송신자는 다음과 같은 구조의 Frame을 네트워크로 보낸다.

```text
[Dst MAC][Src MAC][EtherType][IP Header][TCP Header][TLS/HTTP Data][FCS]
```

이 Frame은 물리 계층에서 전기 신호, 광 신호, 무선 신호 등으로 변환되어 전송된다.

---

## 7. 수신 측 Decapsulation 흐름

수신자는 송신자와 반대 방향으로 데이터를 처리한다.

---

### 7.1 Network Access Layer에서 Frame 수신

수신 장비의 NIC는 Frame을 수신한다.

```text
[Ethernet Header][IP Packet][Ethernet Trailer]
```

이때 다음을 확인한다.

```text
Destination MAC이 내 MAC인가?
Broadcast 또는 Multicast인가?
Frame이 손상되지 않았는가?
EtherType은 무엇인가?
```

EtherType이 IPv4라면 상위 계층으로 IPv4 Packet을 전달한다.  
EtherType이 IPv6라면 IPv6 Packet을 전달한다.  
EtherType이 ARP라면 ARP 처리 루틴으로 전달한다.

---

### 7.2 Internet Layer에서 IP Packet 해석

IP 계층은 IP Header를 확인한다.

```text
[IP Header][TCP Segment or UDP Datagram]
```

확인하는 정보는 다음과 같다.

```text
Destination IP가 내 IP인가?
TTL 또는 Hop Limit이 유효한가?
상위 프로토콜이 TCP인가, UDP인가, ICMP인가?
패킷이 조각화되어 있다면 재조립이 필요한가?
```

목적지 IP가 자신이면 IP Header를 제거하고 상위 계층으로 Payload를 넘긴다.

---

### 7.3 Transport Layer에서 Segment / Datagram 해석

TCP라면 TCP Header를 확인한다.

```text
[TCP Header][Application Data]
```

확인하는 정보는 다음과 같다.

```text
Destination Port가 어떤 프로세스에 연결되는가?
Sequence Number가 올바른가?
Checksum이 유효한가?
TCP 연결 상태와 맞는 Segment인가?
```

UDP라면 UDP Header를 확인한다.

```text
[UDP Header][Application Data]
```

확인하는 정보는 다음과 같다.

```text
Destination Port가 어떤 프로세스에 연결되는가?
Length가 올바른가?
Checksum이 유효한가?
```

Transport 계층은 목적지 포트를 기준으로 데이터를 적절한 애플리케이션 프로세스에 전달한다.

---

### 7.4 Application Layer에서 데이터 처리

Application 계층은 최종 데이터를 해석한다.

HTTP 예시:

```text
GET / HTTP/1.1
Host: www.example.com
```

DNS 예시:

```text
www.example.com A record query
```

SSH 예시:

```text
SSH protocol message
```

Application 계층에서는 다음을 확인할 수 있다.

```text
요청 형식이 올바른가?
인증 정보가 유효한가?
TLS 인증서나 암호화 설정이 올바른가?
HTTP Host Header가 맞는가?
URL 경로가 존재하는가?
서버 애플리케이션이 정상 응답하는가?
```

---

## 8. Segment, Packet, Frame 구분

네트워크 문서나 실무 대화에서는 “패킷”이라는 말을 넓게 사용하는 경우가 많다.  
하지만 정확히 구분하면 다음과 같다.

| 용어 | 주로 관련된 계층 | TCP/IP 관점 | 설명 |
|---|---|---|---|
| Data / Message | Application | Application Layer | 애플리케이션이 만든 데이터 |
| Segment | Transport | TCP | TCP Header가 붙은 데이터 단위 |
| Datagram | Transport / Internet | UDP, IP | UDP 또는 IP에서 자주 쓰는 데이터 단위 |
| Packet | Internet | IP | IP Header가 붙은 데이터 단위 |
| Frame | Network Access | Ethernet, Wi-Fi | MAC Header와 Trailer가 포함될 수 있는 데이터 단위 |
| Bits / Signal | Physical 성격 | 물리 매체 | 전기, 광, 무선 신호 |

---

### 8.1 OSI 관점의 데이터 단위

OSI 관점에서는 각 계층의 데이터 단위를 PDU라고 부른다.

| OSI 계층 | 대표 PDU 표현 |
|---|---|
| Application / Presentation / Session | Data |
| Transport | Segment 또는 Datagram |
| Network | Packet |
| Data Link | Frame |
| Physical | Bits |

---

### 8.2 TCP/IP 관점의 데이터 단위

TCP/IP 4계층 관점에서는 다음처럼 정리할 수 있다.

| TCP/IP 계층 | 데이터 단위 | 예시 |
|---|---|---|
| Application | Data, Message | HTTP Message, DNS Message |
| Transport | TCP Segment, UDP Datagram | TCP Segment, UDP Datagram |
| Internet | IP Packet, IP Datagram | IPv4 Packet, IPv6 Packet |
| Network Access | Frame | Ethernet Frame, Wi-Fi Frame |

---

### 8.3 용어 사용 시 주의점

실무에서는 다음처럼 말하는 경우가 많다.

```text
패킷 캡처해 보세요.
패킷이 안 나가요.
패킷 드롭이 발생했습니다.
```

이때 “패킷”은 정확히 IP Packet만 의미하지 않을 수 있다.  
문맥에 따라 Ethernet Frame, TCP Segment, UDP Datagram까지 넓게 부르는 경우가 있다.

정확한 문서에서는 다음처럼 쓰는 것이 좋다.

```text
TCP Segment
UDP Datagram
IP Packet / IP Datagram
Ethernet Frame
```

---

## 9. 계층별 Encapsulation 구조 예시

TCP 기반 HTTPS의 단순화된 구조는 다음과 같다.

```text
Ethernet Frame
└── Ethernet Header
    ├── Destination MAC
    ├── Source MAC
    └── EtherType: IPv4 or IPv6

└── IP Packet
    └── IP Header
        ├── Source IP
        ├── Destination IP
        ├── TTL or Hop Limit
        └── Protocol or Next Header: TCP

    └── TCP Segment
        └── TCP Header
            ├── Source Port
            ├── Destination Port
            ├── Sequence Number
            ├── Acknowledgment Number
            └── Flags

        └── TLS / HTTP Data

└── Ethernet Trailer
    └── FCS
```

UDP 기반 DNS의 단순화된 구조는 다음과 같다.

```text
Ethernet Frame
└── Ethernet Header
    └── EtherType: IPv4 or IPv6

└── IP Packet
    └── IP Header
        └── Protocol or Next Header: UDP

    └── UDP Datagram
        └── UDP Header
            ├── Source Port
            ├── Destination Port: 53
            ├── Length
            └── Checksum

        └── DNS Message

└── Ethernet Trailer
    └── FCS
```

ARP는 IP Packet 안에 들어가지 않는다.  
ARP는 IPv4에서 같은 링크의 IP 주소를 MAC 주소로 해석하기 위한 프로토콜이며, TCP/IP 구조에서는 보통 Link / Network Access 영역에서 다룬다.

```text
Ethernet Frame
└── Ethernet Header
    └── EtherType: ARP

└── ARP Message
    ├── Sender MAC
    ├── Sender IP
    ├── Target MAC
    └── Target IP

└── Ethernet Trailer
    └── FCS
```

IPv6에서는 ARP 대신 Neighbor Discovery를 사용한다.  
Neighbor Discovery는 ICMPv6 기반으로 동작하며, 같은 링크의 이웃 노드 탐색, 주소 해석, 라우터 탐색, 도달 가능성 확인 등을 처리한다.

---

## 10. 라우터를 지날 때 Encapsulation은 어떻게 바뀌는가?

라우터는 단순히 Frame을 그대로 다음 링크로 전달하지 않는다.

일반적인 라우터 처리 흐름은 다음과 같다.

```text
1. 들어온 Frame을 수신한다.
2. Frame Header와 Trailer를 확인한다.
3. Frame에서 IP Packet을 꺼낸다.
4. IP Header의 Destination IP를 확인한다.
5. TTL 또는 Hop Limit을 감소시킨다.
6. 라우팅 테이블을 보고 다음 홉을 결정한다.
7. 다음 링크에 맞는 새 Frame을 만든다.
8. 새 Frame을 다음 홉으로 전송한다.
```

중요한 점은 다음과 같다.

```text
목적지 IP 주소는 보통 최종 목적지를 유지한다.
하지만 Layer 2 Source MAC / Destination MAC은 링크마다 바뀐다.
IPv4에서는 TTL이 감소하고 Header Checksum이 다시 계산된다.
IPv6에서는 Hop Limit이 감소한다.
```

예시:

```text
PC → Router1 → Router2 → Web Server
```

각 구간에서 MAC 주소는 달라질 수 있다.

```text
PC to Router1:
Source MAC = PC MAC
Destination MAC = Router1 MAC

Router1 to Router2:
Source MAC = Router1 outgoing interface MAC
Destination MAC = Router2 MAC

Router2 to Web Server:
Source MAC = Router2 outgoing interface MAC
Destination MAC = Web Server MAC
```

하지만 일반적인 라우팅 환경에서는 IP 주소는 다음과 같이 유지된다.

```text
Source IP = Client IP
Destination IP = Web Server IP
```

> 예외: NAT, 프록시, 터널링, 로드밸런서, 보안 장비가 개입하면 IP 주소, 포트, 캡슐화 구조가 변경될 수 있다.

---

## 11. 스위치, 라우터, 방화벽이 보는 부분

## 11.1 스위치

스위치는 주로 Network Access 계층의 Frame Header를 본다.

| 확인 정보 | 설명 |
|---|---|
| Source MAC | MAC Address Table 학습 |
| Destination MAC | 어느 포트로 전달할지 결정 |
| VLAN Tag | VLAN 분리와 전달 범위 결정 |
| FCS | 전송 오류 확인에 사용될 수 있음 |

스위치는 일반적으로 IP Header나 TCP Header를 깊게 해석하지 않고도 Frame을 전달할 수 있다.

```text
스위치의 기본 관심사:
Ethernet Header
    ├── Source MAC
    ├── Destination MAC
    └── VLAN Tag
```

> 주의: L3 스위치, 보안 스위치, SDN 장비 등은 IP, Port, ACL 등의 정보를 함께 볼 수 있으므로 “스위치는 무조건 MAC만 본다”고 단정하면 안 된다.

---

## 11.2 라우터

라우터는 주로 Internet 계층의 IP Header를 본다.

| 확인 정보 | 설명 |
|---|---|
| Destination IP | 다음 홉 결정 |
| TTL / Hop Limit | 패킷 생존 시간 제어 |
| Routing Table | 목적지 네트워크로 가는 경로 선택 |
| MTU | 다음 링크로 보낼 수 있는 최대 크기 확인 |

라우터의 기본 관심사는 다음과 같다.

```text
IP Header
    ├── Destination IP
    ├── TTL or Hop Limit
    ├── Protocol or Next Header
    └── Fragmentation 관련 정보
```

라우터는 다음 링크로 보낼 때 새 Layer 2 Frame을 만든다.

---

## 11.3 방화벽

방화벽은 제품, 정책, 모드에 따라 여러 계층을 볼 수 있다.

| 방화벽 유형 | 주로 보는 정보 | 관련 계층 |
|---|---|---|
| Packet Filtering | Source/Destination IP, Protocol, Port | L3/L4 |
| Stateful Firewall | 세션 상태, TCP 상태, UDP/ICMP 흐름 상태 | L3/L4 중심 |
| Application Firewall / WAF | HTTP Host, URL, Method, Header 등 | L7 |
| NGFW | 애플리케이션 식별, 사용자, 콘텐츠 검사 등 | 여러 계층 |

방화벽 정책 예시는 다음과 같다.

```text
Source IP: 10.0.1.10
Destination IP: 203.0.113.10
Protocol: TCP
Destination Port: 443
Action: Allow
```

이 정책은 주로 IP Header와 TCP Header를 기준으로 한다.

반면 다음 정책은 Application 계층 정보가 필요하다.

```text
Host: admin.example.com
URL Path: /admin
HTTP Method: POST
Action: Block
```

HTTPS 트래픽의 Application 내용을 검사하려면 TLS 복호화 설정이 필요할 수 있다.  
실제 가능 여부는 방화벽 제품, 라이선스, 구성, 인증서 배포 방식, 조직 정책에 따라 달라진다.

---

## 12. Wireshark에서 Encapsulation 구조 확인하기

Wireshark는 캡처한 데이터를 계층 구조로 보여준다.  
Encapsulation을 확인할 때는 주로 `Packet List`, `Packet Details`, `Packet Bytes`, `Packet Diagram`을 본다.

---

### 12.1 Packet List

Packet List는 캡처된 패킷 목록을 보여준다.

주로 확인할 항목은 다음과 같다.

```text
No.
Time
Source
Destination
Protocol
Length
Info
```

Packet List에서는 전체 흐름을 본다.

```text
ARP Request / Reply가 있는가?
DNS Query / Response가 있는가?
TCP 3-way handshake가 있는가?
TLS handshake가 진행되는가?
HTTP 요청/응답이 있는가?
ICMP 오류가 보이는가?
```

---

### 12.2 Packet Details

Packet Details는 선택한 패킷의 계층별 구조를 트리 형태로 보여준다.

예를 들어 TCP 기반 HTTPS 트래픽은 다음처럼 보일 수 있다.

```text
Frame
Ethernet II
Internet Protocol Version 4
Transmission Control Protocol
Transport Layer Security
```

또는 DNS 트래픽은 다음처럼 보일 수 있다.

```text
Frame
Ethernet II
Internet Protocol Version 4
User Datagram Protocol
Domain Name System
```

Wireshark에서 Encapsulation을 공부할 때 가장 중요한 영역이 Packet Details이다.

확인할 부분은 다음과 같다.

| Wireshark 항목 | 확인 내용 |
|---|---|
| Frame | 캡처 길이, 실제 프레임 관련 정보 |
| Ethernet II | Source MAC, Destination MAC, EtherType |
| IPv4 / IPv6 | Source IP, Destination IP, TTL/Hop Limit, Protocol |
| TCP / UDP | Source Port, Destination Port, Flags, Sequence Number |
| Application Protocol | DNS, HTTP, TLS 등 상위 프로토콜 내용 |

---

### 12.3 Packet Bytes

Packet Bytes는 실제 캡처된 바이트를 Hex dump 형태로 보여준다.

Packet Details에서 특정 Header 필드를 클릭하면 Packet Bytes에서 해당 바이트 위치가 강조된다.  
이 기능을 사용하면 Header가 실제 바이트 배열에서 어디에 위치하는지 확인할 수 있다.

예를 들어 Ethernet Header는 일반적으로 Frame 앞쪽에 위치한다.

```text
Destination MAC: 6 bytes
Source MAC:      6 bytes
EtherType:       2 bytes
```

IPv4 Header는 그 뒤에 이어진다.

```text
Version / IHL
DSCP / ECN
Total Length
Identification
Flags / Fragment Offset
TTL
Protocol
Header Checksum
Source IP
Destination IP
```

TCP Header는 IP Header 뒤에 이어진다.

```text
Source Port
Destination Port
Sequence Number
Acknowledgment Number
Flags
Window Size
Checksum
```

---

### 12.4 Packet Diagram

Wireshark 버전에 따라 Packet Diagram Pane을 사용할 수 있다.  
Packet Diagram은 선택한 패킷의 프로토콜과 주요 필드를 교재나 RFC 그림처럼 다이어그램 형태로 보여준다.

Encapsulation 구조를 시각적으로 이해할 때 유용하다.

```text
Ethernet Header
IP Header
TCP Header
Application Data
```

다만 모든 환경에서 동일하게 보이는 것은 아니며, Wireshark 버전과 레이아웃 설정에 따라 표시 방식이 다를 수 있다.

---

### 12.5 Wireshark에서 자주 보는 Display Filter

Encapsulation 구조를 확인할 때 자주 사용하는 필터는 다음과 같다.

```text
arp
icmp
icmpv6
dns
tcp
udp
tcp.port == 443
udp.port == 53
ip.addr == 203.0.113.10
eth.addr == aa:bb:cc:dd:ee:ff
http
tls
```

TCP 3-way handshake를 볼 때는 다음 필터가 유용하다.

```text
tcp.flags.syn == 1
tcp.flags.ack == 1
```

특정 호스트와의 통신만 보고 싶다면 다음처럼 필터링할 수 있다.

```text
ip.addr == 192.0.2.10
```

특정 MAC 주소를 보고 싶다면 다음처럼 필터링할 수 있다.

```text
eth.addr == 00:11:22:33:44:55
```

---

### 12.6 Wireshark 확인 시 주의사항

Wireshark에서 보이는 내용이 항상 물리 매체 위의 원본과 완전히 같지는 않을 수 있다.

주의할 점은 다음과 같다.

| 항목 | 설명 |
|---|---|
| Ethernet FCS | 일반 PC 캡처에서는 보이지 않는 경우가 많다. |
| Checksum Offloading | NIC가 체크섬 계산을 대신하면 송신 측 캡처에서 checksum incorrect처럼 보일 수 있다. |
| TLS 암호화 | 복호화 키가 없으면 HTTP 내용이 보이지 않는다. |
| VLAN Tag | NIC, 드라이버, OS 설정에 따라 VLAN Tag가 보이지 않을 수 있다. |
| 캡처 위치 | 클라이언트, 서버, 스위치 SPAN, 라우터 등 위치에 따라 보이는 Header가 다르다. |
| NAT 이후 캡처 | NAT 전후에서 Source IP 또는 Port가 다르게 보일 수 있다. |

---

## 13. 장애 분석에서 Encapsulation / Decapsulation이 중요한 이유

Encapsulation / Decapsulation을 이해하면 “어느 계층에서 문제가 생겼는지”를 좁힐 수 있다.

---

### 13.1 Frame이 보이지 않는 경우

증상:

```text
패킷 캡처를 했는데 아무것도 보이지 않는다.
```

가능한 원인:

```text
NIC가 down 상태이다.
케이블 또는 Wi-Fi 연결이 끊겼다.
잘못된 인터페이스에서 캡처하고 있다.
SPAN/Mirror 설정이 잘못되었다.
VLAN이 다르다.
```

확인 명령어:

Linux:

```bash
ip link
ip addr
sudo tcpdump -ni <interface>
```

Windows:

```powershell
ipconfig /all
Get-NetAdapter
```

---

### 13.2 ARP가 실패하는 경우

증상:

```text
같은 네트워크의 게이트웨이에 접근할 수 없다.
```

Wireshark에서 볼 수 있는 모습:

```text
ARP Request는 반복되지만 ARP Reply가 없다.
```

가능한 원인:

```text
기본 게이트웨이 IP가 잘못되었다.
VLAN이 다르다.
스위치 포트 설정이 잘못되었다.
대상 장비가 down 상태이다.
ARP 차단 또는 보안 정책이 있다.
```

확인 명령어:

Linux:

```bash
ip neigh
sudo tcpdump -ni <interface> arp
```

Windows:

```powershell
arp -a
```

---

### 13.3 IP Packet은 나가지만 응답이 없는 경우

증상:

```text
ARP는 되지만 외부 IP로 ping이 되지 않는다.
```

가능한 원인:

```text
기본 게이트웨이가 잘못되었다.
라우팅 테이블에 경로가 없다.
중간 방화벽에서 차단한다.
상대방이 ICMP를 차단한다.
NAT 또는 ACL 문제가 있다.
```

확인 명령어:

Linux:

```bash
ip route
ip route get <destination-ip>
ping -c 4 <destination-ip>
traceroute <destination-ip>
```

Windows:

```powershell
route print
ping <destination-ip>
tracert <destination-ip>
Test-NetConnection <destination-host> -TraceRoute
```

---

### 13.4 TCP 연결이 실패하는 경우

증상:

```text
DNS 해석은 되지만 웹사이트 접속이 안 된다.
```

Wireshark에서 볼 수 있는 모습:

```text
SYN만 반복되고 SYN-ACK이 없다.
SYN 이후 RST가 온다.
3-way handshake 이후 TLS handshake가 실패한다.
```

가능한 원인:

```text
서버 포트가 열려 있지 않다.
방화벽이 TCP 포트를 차단한다.
서버 애플리케이션이 down 상태이다.
로드밸런서 또는 보안 장비가 차단한다.
TLS 정책이 맞지 않는다.
```

확인 명령어:

Linux:

```bash
ss -lntup
nc -vz <host> 443
curl -vI https://<host>/
sudo tcpdump -ni <interface> tcp port 443
```

Windows:

```powershell
Test-NetConnection <host> -Port 443 -InformationLevel Detailed
netstat -ano
```

---

### 13.5 UDP 통신 확인이 어려운 경우

UDP는 TCP처럼 연결 상태가 명확하지 않다.  
따라서 UDP 장애 분석에서는 요청과 응답을 패킷 캡처로 함께 확인하는 것이 중요하다.

DNS 예시:

```text
Client → DNS Server: UDP Query
DNS Server → Client: UDP Response
```

가능한 원인:

```text
UDP 포트가 방화벽에서 차단되었다.
응답 패킷이 돌아오지 않는다.
DNS 서버가 응답하지 않는다.
응답 크기 문제로 TCP fallback이 필요하다.
```

확인 명령어:

Linux:

```bash
dig www.example.com
dig @8.8.8.8 www.example.com
sudo tcpdump -ni <interface> udp port 53
```

Windows:

```powershell
nslookup www.example.com
Resolve-DnsName www.example.com
```

---

### 13.6 Application 계층 문제인 경우

하위 계층 통신은 정상인데 애플리케이션이 실패할 수도 있다.

증상:

```text
TCP 연결은 된다.
TLS handshake도 된다.
하지만 HTTP 403, 404, 500 오류가 발생한다.
```

가능한 원인:

```text
Host Header가 잘못되었다.
URL 경로가 잘못되었다.
인증 또는 권한 문제가 있다.
서버 애플리케이션 오류가 있다.
Reverse Proxy 설정이 잘못되었다.
```

확인 명령어:

Linux / Windows:

```bash
curl -vI https://www.example.com/
curl -v https://www.example.com/
```

---

## 14. MTU, MSS, Fragmentation과 Encapsulation

Encapsulation을 이해할 때 MTU와 MSS도 함께 알아야 한다.

---

### 14.1 MTU

MTU(Maximum Transmission Unit)는 한 링크에서 전달할 수 있는 최대 Packet 크기를 의미한다.

Ethernet에서 일반적으로 사용하는 MTU는 1500 bytes이다.  
다만 Jumbo Frame, 터널링, VPN, 클라우드 네트워크 환경에서는 값이 달라질 수 있다.

```text
MTU = IP Packet이 링크 계층 Payload로 들어갈 수 있는 최대 크기
```

Encapsulation으로 Header가 추가되면 실제 애플리케이션 데이터가 사용할 수 있는 공간은 줄어든다.

---

### 14.2 MSS

MSS(Maximum Segment Size)는 TCP Payload로 실을 수 있는 최대 데이터 크기를 의미한다.

일반적인 IPv4 over Ethernet 환경에서는 다음처럼 계산할 수 있다.

```text
Ethernet MTU 1500 bytes
- IPv4 Header 20 bytes
- TCP Header 20 bytes
= TCP MSS 1460 bytes
```

단, TCP Options, IPv6, 터널링, VPN, PPPoE 등이 있으면 MSS 값은 달라질 수 있다.

---

### 14.3 Fragmentation

IP Packet이 경로의 MTU보다 크면 Fragmentation 또는 Packet Drop이 발생할 수 있다.

IPv4에서는 조건에 따라 라우터 또는 송신자가 Fragmentation을 수행할 수 있다.  
IPv6에서는 중간 라우터가 Fragmentation을 수행하지 않고, 송신자가 Path MTU Discovery를 통해 적절한 크기를 사용해야 한다.

장애 관점에서는 다음 문제가 발생할 수 있다.

```text
작은 ping은 되지만 큰 데이터 전송이 실패한다.
VPN 연결 후 특정 사이트만 느리거나 안 된다.
TLS handshake 중간에 멈춘다.
파일 업로드가 실패한다.
```

확인 포인트:

```text
MTU 값
DF bit 설정
ICMP Fragmentation Needed 메시지
Path MTU Discovery 차단 여부
터널링 Header 추가 여부
```

---

## 15. NAT, 터널링, VLAN과 Encapsulation

기본 Encapsulation 구조는 실제 네트워크에서 NAT, 터널링, VLAN 같은 기술에 의해 변형될 수 있다.

---

### 15.1 NAT

NAT는 IP 주소 또는 포트 정보를 변경한다.

예시:

```text
Before NAT:
Source IP: 10.0.1.10
Source Port: 51524

After NAT:
Source IP: 198.51.100.10
Source Port: 40001
```

NAT 장비 전후에서 패킷을 캡처하면 IP Header와 TCP/UDP Header가 다르게 보일 수 있다.

---

### 15.2 터널링

터널링은 기존 패킷을 또 다른 패킷 안에 넣는 방식이다.

예시:

```text
Original IP Packet
    ↓
Tunnel Header 추가
    ↓
Outer IP Packet
```

VPN, GRE, VXLAN, IPsec, WireGuard 등은 각기 다른 방식으로 캡슐화를 추가한다.

터널링 환경에서는 다음을 구분해야 한다.

```text
Inner Header: 원래 통신의 Header
Outer Header: 터널을 통과하기 위한 Header
```

---

### 15.3 VLAN Tag

VLAN을 사용하는 Ethernet 환경에서는 Frame에 802.1Q Tag가 들어갈 수 있다.

```text
[Dst MAC][Src MAC][802.1Q Tag][EtherType][Payload][FCS]
```

VLAN Tag에는 VLAN ID와 우선순위 정보가 포함된다.  
스위치에서 VLAN이 잘못 설정되면 Frame이 올바른 네트워크로 전달되지 않을 수 있다.

---

## 16. 실무 분석 순서

Encapsulation / Decapsulation 관점에서 장애를 분석할 때는 아래 순서를 사용할 수 있다.

---

### 16.1 1단계: 링크와 Frame 확인

```text
NIC가 up 상태인가?
올바른 인터페이스에서 캡처하고 있는가?
Frame이 들어오고 나가는가?
VLAN이 맞는가?
```

명령어:

```bash
ip link
ip addr
sudo tcpdump -ni <interface>
```

---

### 16.2 2단계: ARP / NDP 확인

```text
기본 게이트웨이 MAC 주소를 알고 있는가?
ARP Request에 Reply가 오는가?
IPv6 환경에서는 Neighbor Solicitation / Advertisement가 보이는가?
```

명령어:

```bash
ip neigh
sudo tcpdump -ni <interface> arp
sudo tcpdump -ni <interface> icmp6
```

---

### 16.3 3단계: IP와 라우팅 확인

```text
내 IP 주소가 맞는가?
Subnet이 맞는가?
Default Gateway가 있는가?
목적지까지 라우팅 경로가 있는가?
```

명령어:

```bash
ip addr
ip route
ip route get <destination-ip>
ping <destination-ip>
traceroute <destination-ip>
```

---

### 16.4 4단계: Transport 확인

```text
TCP 3-way handshake가 되는가?
SYN에 대한 SYN-ACK이 오는가?
RST가 오는가?
UDP 요청과 응답이 모두 보이는가?
```

명령어:

```bash
nc -vz <host> 443
ss -ant
sudo tcpdump -ni <interface> tcp port 443
sudo tcpdump -ni <interface> udp port 53
```

---

### 16.5 5단계: Application 확인

```text
DNS 응답이 정상인가?
TLS handshake가 성공하는가?
HTTP 상태 코드는 무엇인가?
서버 로그에는 어떤 오류가 있는가?
```

명령어:

```bash
dig <domain>
curl -vI https://<domain>/
curl -v https://<domain>/
```

---

## 17. 명령어 요약

| 목적 | Linux | Windows |
|---|---|---|
| 인터페이스 확인 | `ip link`, `ip addr` | `ipconfig /all`, `Get-NetAdapter` |
| ARP/NDP 확인 | `ip neigh` | `arp -a` |
| 라우팅 확인 | `ip route`, `ip route get <ip>` | `route print` |
| ICMP 확인 | `ping`, `traceroute` | `ping`, `tracert` |
| TCP 포트 확인 | `nc -vz <host> <port>`, `ss -ant` | `Test-NetConnection <host> -Port <port>` |
| DNS 확인 | `dig`, `nslookup` | `nslookup`, `Resolve-DnsName` |
| HTTP/TLS 확인 | `curl -vI`, `curl -v` | `curl.exe -vI`, `curl.exe -v` |
| 패킷 캡처 | `tcpdump`, Wireshark | Wireshark, pktmon |

> 명령어 옵션은 OS, 배포판, 버전, 권한, 설치 패키지에 따라 다를 수 있다.  
> 운영 환경에서는 공식 문서, `man`, `--help`, 벤더 문서를 함께 확인해야 한다.

---

## 18. 정리

Encapsulation은 송신자가 데이터를 네트워크로 보내기 위해 계층별 Header와 Trailer를 붙이는 과정이다.

```text
Application Data
    ↓
Transport Header 추가
    ↓
IP Header 추가
    ↓
Frame Header / Trailer 추가
    ↓
전송
```

Decapsulation은 수신자가 받은 Frame을 계층별로 해석하고 최종 애플리케이션 데이터로 복원하는 과정이다.

```text
Frame 수신
    ↓
IP Packet 추출
    ↓
TCP Segment 또는 UDP Datagram 추출
    ↓
Application Data 추출
```

핵심 구분은 다음과 같다.

```text
Segment = Transport 계층, 주로 TCP
Packet  = Internet / Network 계층, 주로 IP
Frame   = Network Access / Data Link 계층, Ethernet 또는 Wi-Fi
```

장애 분석에서는 다음 흐름으로 확인하는 것이 좋다.

```text
Frame이 보이는가?
ARP 또는 NDP가 되는가?
IP 라우팅이 되는가?
TCP/UDP 통신이 되는가?
Application 응답이 정상인가?
```

Encapsulation / Decapsulation을 이해하면 Wireshark에서 보이는 계층 구조를 읽을 수 있고, 네트워크 장애를 감으로 추측하지 않고 계층별로 좁혀갈 수 있다.

---

## 19. 확실하지 않거나 환경 의존적인 부분

다음 내용은 환경에 따라 달라질 수 있다.

| 항목 | 설명 |
|---|---|
| TCP/IP 계층 수 | 문헌에 따라 4계층 또는 5계층으로 설명될 수 있다. 이 문서는 `TCP_IP_Model.md`와 맞춰 4계층 구조를 사용한다. |
| Wireshark 표시 항목 | NIC, 드라이버, OS, 오프로딩 설정, 캡처 위치에 따라 표시되는 Header와 Checksum 정보가 달라질 수 있다. |
| Ethernet FCS 표시 | 일반적인 엔드포인트 캡처에서는 FCS가 보이지 않을 수 있다. |
| VLAN Tag 표시 | OS, NIC, 드라이버 설정에 따라 Wireshark에서 VLAN Tag가 보이지 않을 수 있다. |
| 방화벽 검사 범위 | 제품, 라이선스, 정책, TLS 복호화 여부에 따라 L3/L4 중심일 수도 있고 L7까지 검사할 수도 있다. |
| NAT / 터널링 | 장비와 구성에 따라 IP, Port, Header 구조가 달라질 수 있다. |
| MTU / MSS | Ethernet, VPN, 클라우드, 터널링, PPPoE, Jumbo Frame 환경에 따라 값이 달라질 수 있다. |

---

## 20. 참고 자료

[1]: https://datatracker.ietf.org/doc/html/rfc1122 "RFC 1122 - Requirements for Internet Hosts -- Communication Layers"  
[2]: https://datatracker.ietf.org/doc/html/rfc1123 "RFC 1123 - Requirements for Internet Hosts -- Application and Support"  
[3]: https://www.iso.org/standard/20269.html "ISO/IEC 7498-1 - OSI Basic Reference Model"  
[4]: https://datatracker.ietf.org/doc/html/rfc791 "RFC 791 - Internet Protocol"  
[5]: https://datatracker.ietf.org/doc/html/rfc8200 "RFC 8200 - Internet Protocol, Version 6"  
[6]: https://datatracker.ietf.org/doc/html/rfc9293 "RFC 9293 - Transmission Control Protocol"  
[7]: https://datatracker.ietf.org/doc/html/rfc768 "RFC 768 - User Datagram Protocol"  
[8]: https://datatracker.ietf.org/doc/html/rfc826 "RFC 826 - Address Resolution Protocol"  
[9]: https://datatracker.ietf.org/doc/html/rfc4861 "RFC 4861 - Neighbor Discovery for IPv6"  
[10]: https://datatracker.ietf.org/doc/html/rfc792 "RFC 792 - Internet Control Message Protocol"  
[11]: https://datatracker.ietf.org/doc/html/rfc4443 "RFC 4443 - ICMPv6"  
[12]: https://datatracker.ietf.org/doc/html/rfc9000 "RFC 9000 - QUIC: A UDP-Based Multiplexed and Secure Transport"  
[13]: https://datatracker.ietf.org/doc/html/rfc9110 "RFC 9110 - HTTP Semantics"  
[14]: https://datatracker.ietf.org/doc/html/rfc9114 "RFC 9114 - HTTP/3"  
[15]: https://standards.ieee.org/ieee/802.3/10422/ "IEEE 802.3-2022 - Standard for Ethernet"  
[16]: https://www.wireshark.org/docs/wsug_html_chunked/ChUsePacketDetailsPaneSection.html "Wireshark User's Guide - Packet Details Pane"  
[17]: https://www.wireshark.org/docs/wsug_html_chunked/ChUsePacketBytesPaneSection.html "Wireshark User's Guide - Packet Bytes Pane"  
[18]: https://www.wireshark.org/docs/wsug_html_chunked/ChUsePacketDiagramPaneSection.html "Wireshark User's Guide - Packet Diagram Pane"  
[19]: https://wiki.wireshark.org/Ethernet "Wireshark Wiki - Ethernet"  
[20]: https://man7.org/linux/man-pages/man8/ip.8.html "Linux man-pages - ip"  
[21]: https://man7.org/linux/man-pages/man8/ss.8.html "Linux man-pages - ss"  
[22]: https://manpages.ubuntu.com/manpages/jammy/man8/tcpdump.8.html "Ubuntu Manpage - tcpdump"  
[23]: https://manpages.ubuntu.com/manpages/jammy/man8/traceroute.8.html "Ubuntu Manpage - traceroute"  
[24]: https://learn.microsoft.com/windows-server/administration/windows-commands/ipconfig "Microsoft - ipconfig"  
[25]: https://learn.microsoft.com/windows-server/administration/windows-commands/arp "Microsoft - arp"  
[26]: https://learn.microsoft.com/windows-server/administration/windows-commands/route_ws2008 "Microsoft - route"  
[27]: https://learn.microsoft.com/powershell/module/nettcpip/test-netconnection "Microsoft - Test-NetConnection"  
[28]: https://learn.microsoft.com/windows-server/administration/windows-commands/netstat "Microsoft - netstat"
