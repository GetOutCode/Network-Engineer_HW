# TCP/IP Model

> 경로: `Network/01_Theory/01_Network_Fundamentals/TCP_IP_Model.md`  
> 연결 문서: [`OSI_7_Layers.md`](./OSI_7_Layers.md)  
> 학습 흐름: `OSI 7계층` → `TCP/IP Model` → `Encapsulation` → `IP Address / Subnet` → `ARP / ICMP / DNS`

---

## 1. 학습 목표

이 문서의 목표는 TCP/IP 모델을 단순 암기용 계층표가 아니라 **실제 인터넷 통신 구조**로 이해하는 것이다.

- OSI 7계층과 TCP/IP 모델의 차이를 이해한다.
- 실제 인터넷 통신에서 TCP/IP 모델이 어떻게 사용되는지 이해한다.
- Encapsulation / Decapsulation, IP Address, Subnet, ARP, ICMP, DNS 학습으로 자연스럽게 연결한다.
- 프레임, 패킷, 세그먼트의 차이를 계층 관점에서 구분한다.
- 스위치, 라우터, 방화벽이 통신 흐름에서 어느 지점에 개입하는지 이해한다.
- 장애 발생 시 계층별로 확인할 수 있는 포인트와 명령어를 정리한다.

---

## 2. 핵심 요약

TCP/IP 모델은 인터넷에서 실제로 사용되는 프로토콜 묶음을 계층적으로 이해하기 위한 모델이다.

```text
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

실무에서는 “OSI 7계층을 TCP/IP 4계층에 억지로 1:1 매칭”하기보다 다음 관점이 더 중요하다.

- 애플리케이션은 **무슨 데이터를 주고받을지** 정한다.
- 전송 계층은 **어떤 프로세스끼리 통신할지**를 포트로 구분한다.
- 인터넷 계층은 **어떤 IP 주소로 보낼지**를 결정한다.
- 네트워크 접근 계층은 **같은 링크 또는 다음 홉까지 어떤 프레임으로 전달할지**를 처리한다.

---

## 3. TCP/IP 모델이 필요한 이유

인터넷은 하나의 거대한 단일 네트워크가 아니다.  
서로 다른 LAN, Wi-Fi, 데이터센터, ISP, 해저 케이블, 라우터 네트워크가 연결된 **네트워크들의 네트워크**이다.

TCP/IP 모델이 필요한 이유는 다음과 같다.

---

### 3.1 서로 다른 네트워크를 하나의 인터넷처럼 연결하기 위해

이더넷, Wi-Fi, 광회선, LTE/5G 등 실제 전송 매체는 서로 다를 수 있다.  
하지만 IP 계층이 공통의 주소 체계와 라우팅 방식을 제공하면, 사용자는 물리 매체의 차이를 몰라도 인터넷 통신을 할 수 있다.

예를 들어 사용자는 다음처럼 접속한다.

```text
https://www.example.com
```

사용자는 이 요청이 Wi-Fi 프레임, 이더넷 프레임, 라우터의 IP 패킷 전달, TCP 연결, UDP 기반 QUIC 연결, TLS 암호화, HTTP 요청 등으로 나뉘어 처리되는지 직접 알 필요가 없다.

---

### 3.2 통신 기능을 역할별로 나누기 위해

TCP/IP 모델은 통신 기능을 여러 계층으로 나눈다.

| 계층 | 주된 관심사 |
|---|---|
| Application | 사용자가 원하는 서비스와 데이터 형식 |
| Transport | 애플리케이션 프로세스 간 통신 |
| Internet | IP 주소 기반의 네트워크 간 전달 |
| Network Access | 같은 링크 또는 다음 홉까지의 실제 프레임 전달 |

이렇게 나누면 한 계층의 기술이 바뀌어도 다른 계층을 그대로 유지할 수 있다.

예를 들어 HTTP는 TCP 위에서도 동작할 수 있고, HTTP/3처럼 QUIC 위에서도 동작할 수 있다.  
또 IP는 이더넷, Wi-Fi 등 서로 다른 링크 기술 위에서 동작할 수 있다.

---

### 3.3 장애를 계층별로 분리해서 확인하기 위해

웹사이트 접속이 안 될 때 원인은 다양하다.

- 케이블 또는 Wi-Fi 문제
- IP 주소 설정 문제
- 기본 게이트웨이 문제
- DNS 문제
- 방화벽 차단
- TCP 포트 미오픈
- UDP 기반 QUIC 차단
- TLS 인증서 문제
- HTTP 애플리케이션 오류

TCP/IP 모델을 사용하면 문제를 다음처럼 나눠 볼 수 있다.

```text
1. 링크가 살아 있는가?
2. IP 주소와 라우팅이 정상인가?
3. TCP/UDP 포트 통신이 가능한가?
4. DNS, TLS, HTTP 애플리케이션 처리가 정상인가?
```

---

## 4. OSI 7계층과 TCP/IP 모델의 차이

## 4.1 OSI 7계층

OSI 7계층은 네트워크 통신을 설명하기 위한 **참조 모델(reference model)** 성격이 강하다.  
즉, 실제 인터넷이 반드시 OSI 7계층 구조 그대로 구현되어야 한다는 의미가 아니다.

| OSI 계층 | 이름 |
|---:|---|
| 7 | Application |
| 6 | Presentation |
| 5 | Session |
| 4 | Transport |
| 3 | Network |
| 2 | Data Link |
| 1 | Physical |

OSI 모델은 통신 기능을 세밀하게 분리해서 설명하는 데 유용하다.  
따라서 학습, 문서화, 장애 분석에서 여전히 많이 사용된다.

---

## 4.2 TCP/IP 모델

TCP/IP 모델은 실제 인터넷 프로토콜 묶음을 설명하는 데 더 직접적이다.

| TCP/IP 계층 | 대표 역할 |
|---|---|
| Application | HTTP, DNS, TLS, SSH 등 애플리케이션 프로토콜과 보안 기술 |
| Transport | TCP, UDP, QUIC 등 애플리케이션 간 전송 |
| Internet | IPv4, IPv6, ICMP 등 IP 기반 전달 |
| Network Access | Ethernet, Wi-Fi, ARP 등 링크 접근 |

> 참고: TCP/IP 모델은 문헌에 따라 4계층, 5계층 등으로 설명되기도 한다.  
> 이 문서에서는 학습 목표에 맞춰 **Application, Transport, Internet, Network Access**의 4계층 구조를 사용한다.

---

## 4.3 OSI와 TCP/IP를 억지로 1:1 매칭하면 안 되는 이유

OSI와 TCP/IP는 목적이 다르다.

| 구분 | OSI 7계층 | TCP/IP 모델 |
|---|---|---|
| 성격 | 참조 모델 | 실제 인터넷 프로토콜 구조 설명 |
| 목적 | 통신 기능을 개념적으로 세분화 | 인터넷 통신을 구현 가능한 구조로 설명 |
| 계층 수 | 7계층 | 보통 4계층 또는 5계층으로 설명 |
| 실무 활용 | 장애 분석, 교육, 개념 설명 | 실제 패킷 흐름, 프로토콜 분석 |
| 구현과의 관계 | 반드시 그대로 구현되는 구조는 아님 | 운영체제, 라우터, 서버, 클라이언트에서 직접 사용 |

특히 다음 개념은 1:1 매칭이 어렵다.

| 개념 | 설명 |
|---|---|
| TLS | OSI식으로는 Presentation 계층처럼 보일 수 있지만, TCP/IP에서는 보통 애플리케이션 데이터를 보호하는 Application 영역의 보안 프로토콜로 다룬다. |
| DNS | Application 계층 프로토콜이지만 실제 질의는 UDP, TCP, TLS, HTTPS 등을 통해 전달될 수 있다. |
| ARP | IPv4에서 IP 주소를 같은 링크의 MAC 주소로 해석하기 위한 프로토콜이다. IP 주소를 사용하지만 IP 패킷을 전달하는 프로토콜 자체는 아니며, TCP/IP 구조에서는 보통 Link / Network Access 영역에서 다룬다. |
| NDP | IPv6에서 ARP 대신 사용되는 Neighbor Discovery는 ICMPv6 기반으로 동작한다. 같은 링크의 이웃 노드 탐색, 주소 해석, 라우터 탐색, 도달 가능성 확인 등을 처리한다. |
| ICMP | 사용자 애플리케이션 데이터는 아니지만 IP 계층의 오류 보고와 진단에 사용된다. |
| QUIC | UDP 위에서 동작하지만 연결 관리, 신뢰성, 흐름 제어, 암호화 기능을 제공하므로 단순히 “UDP 애플리케이션”으로만 보면 부족하다. |

따라서 실무에서는 다음처럼 이해하는 것이 좋다.

```text
OSI 7계층 = 통신 기능을 설명하기 위한 세밀한 개념 지도
TCP/IP 모델 = 실제 인터넷 통신 흐름을 이해하기 위한 구조
```

---

## 5. TCP/IP 모델의 계층별 역할

## 5.1 Application Layer

Application 계층은 사용자가 실제로 이용하는 서비스와 데이터 형식을 정의한다.

| 항목 | 설명 |
|---|---|
| 역할 | 애플리케이션 간 데이터 형식, 요청/응답 규칙, 인증, 암호화, 이름 해석 등을 처리 |
| 대표 프로토콜 / 기술 | HTTP, DNS, SSH, SMTP, IMAP, TLS, HTTP over TLS(HTTPS) |
| 주소 관점 | URL, 도메인 이름, 애플리케이션 식별자 |
| 데이터 단위 | Data, Message |
| 예시 | 브라우저가 `GET /index.html` HTTP 요청을 생성 |

Application 계층의 예시는 다음과 같다.

```text
HTTP 요청:
GET / HTTP/1.1
Host: www.example.com
```

이 요청은 그 자체로는 네트워크를 건널 수 없다.  
아래 계층들이 이 데이터를 목적지까지 운반한다.

HTTPS는 별도의 완전히 새로운 애플리케이션 프로토콜이라기보다, 일반적으로 HTTP를 TLS로 보호해서 사용하는 형태로 이해하는 것이 좋다.  
HTTP/1.1 over TLS 또는 HTTP/2 over TLS는 일반적으로 TCP 443을 사용한다.  
다만 HTTP/3은 QUIC 위에서 동작하며, QUIC은 UDP 기반 전송 프로토콜이다. 따라서 HTTP/3 접속은 일반적으로 UDP 443을 사용하거나, 서버가 광고한 대체 UDP 포트를 사용할 수 있다.

---

## 5.2 Transport Layer

Transport 계층은 호스트 안의 **프로세스와 프로세스 간 통신**을 담당한다.

| 항목 | 설명 |
|---|---|
| 역할 | 포트 번호를 사용해 애플리케이션 프로세스를 구분 |
| 대표 프로토콜 | TCP, UDP, QUIC |
| 주소 관점 | Source Port, Destination Port |
| 데이터 단위 | TCP Segment, UDP Datagram, QUIC Packet |
| 예시 | 클라이언트 임시 포트에서 서버의 TCP 443 또는 UDP 443으로 연결 |

TCP와 UDP의 차이는 다음과 같다.

| 구분 | TCP | UDP |
|---|---|---|
| 연결 개념 | 연결 지향 | 비연결형 |
| 신뢰성 | 재전송, 순서 보장, 흐름 제어 제공 | UDP 자체는 전달 보장, 순서 보장, 중복 제거를 제공하지 않음 |
| 데이터 단위 | Segment | Datagram |
| 대표 사용 | HTTP/1.1, HTTP/2, SSH 등 | DNS, DHCP, QUIC 기반 HTTP/3 등 |

TCP는 애플리케이션에 신뢰성 있는 순서화된 바이트 스트림을 제공한다.  
애플리케이션의 바이트 스트림은 TCP 세그먼트로 나뉘고, 각 TCP 세그먼트는 IP 데이터그램에 실려 네트워크로 전달된다.

UDP는 TCP보다 단순한 전송 방식을 제공한다.  
UDP 자체는 연결 설정, 재전송, 순서 보장, 흐름 제어를 제공하지 않는다.  
다만 UDP를 사용한다고 해서 항상 애플리케이션이 신뢰성을 전혀 제공하지 않는다는 뜻은 아니다. QUIC처럼 UDP 위에서 별도의 연결 관리, 신뢰성, 암호화 기능을 구현하는 프로토콜도 있다.

---

## 5.3 Internet Layer

Internet 계층은 IP 주소를 기반으로 패킷을 목적지 네트워크까지 전달한다.

| 항목 | 설명 |
|---|---|
| 역할 | IP 주소 지정, 라우팅, 패킷 전달, 오류 보고 보조 |
| 대표 프로토콜 | IPv4, IPv6, ICMP, ICMPv6 |
| 주소 관점 | Source IP Address, Destination IP Address |
| 데이터 단위 | IP Packet, IP Datagram |
| 예시 | `192.0.2.10`에서 `203.0.113.20`으로 IP 패킷 전송 |

Internet 계층의 핵심은 다음과 같다.

```text
목적지 IP가 내 로컬 네트워크에 있는가?
    ├─ 예: 같은 링크의 대상에게 직접 전달
    └─ 아니오: 기본 게이트웨이 또는 라우터로 전달
```

라우터는 IP 패킷의 목적지 IP 주소를 보고 다음 홉을 결정한다.  
다만 라우터를 지날 때 IP 패킷의 모든 필드가 그대로 유지되는 것은 아니다.

라우터를 지날 때 목적지 IP 주소는 보통 최종 목적지를 유지한다.  
다만 IPv4에서는 TTL이 감소하고, 이로 인해 IPv4 Header Checksum이 다시 계산된다.  
IPv6에서는 Hop Limit이 감소한다.  
또한 다음 링크로 나갈 때 Layer 2 Frame은 다음 홉에 맞게 새로 만들어진다.

> 예외: NAT, 터널링, 프록시, 보안 장비가 개입하는 경우 IP 주소, 포트, 페이로드 처리 방식이 달라질 수 있다.

---

## 5.4 Network Access Layer

Network Access 계층은 같은 링크 또는 다음 홉까지 실제로 데이터를 전달한다.

| 항목 | 설명 |
|---|---|
| 역할 | 프레임 생성, MAC 주소 기반 전달, 물리 매체 접근 |
| 대표 기술 | Ethernet, Wi-Fi |
| 관련 프로토콜 | ARP |
| IPv6 관련 보조 메커니즘 | Neighbor Discovery |
| 주소 관점 | MAC Address |
| 데이터 단위 | Frame |
| 예시 | 내 PC가 기본 게이트웨이의 MAC 주소로 Ethernet Frame 전송 |

Network Access 계층에서는 다음과 같은 일이 발생한다.

```text
IP 패킷을 Ethernet Frame 안에 넣는다.
Frame의 목적지 MAC 주소를 다음 홉 장비의 MAC 주소로 설정한다.
```

중요한 점은 다음과 같다.

```text
목적지 IP 주소 = 최종 목적지
목적지 MAC 주소 = 현재 링크에서의 다음 홉
```

예를 들어 웹 서버가 다른 네트워크에 있다면, 내 PC가 처음 보내는 프레임의 목적지 MAC 주소는 웹 서버의 MAC 주소가 아니라 **기본 게이트웨이의 MAC 주소**이다.

IPv4에서는 같은 링크의 다음 홉 MAC 주소를 찾기 위해 일반적으로 ARP를 사용한다.  
IPv6에서는 ARP 대신 Neighbor Discovery를 사용한다. Neighbor Discovery는 ICMPv6 기반으로 동작하며, 이웃 노드 탐색, 주소 해석, 라우터 탐색, 도달 가능성 확인 등을 처리한다.

---

## 6. 웹사이트 접속 시 TCP/IP 통신 흐름

예시 상황은 다음과 같다.

```text
사용자 PC → https://www.example.com 접속
```

실제 흐름은 구현, 브라우저, 운영체제, 서버 설정, 네트워크 정책에 따라 달라질 수 있다.  
아래는 일반적인 HTTPS 접속 흐름이다.

---

## 6.1 1단계: URL 해석

브라우저는 URL을 해석한다.

```text
https://www.example.com/
```

| 구성 요소 | 의미 |
|---|---|
| `https` | 사용할 스킴 |
| `www.example.com` | 접속할 호스트 이름 |
| `/` | 요청 경로 |
| 일반적인 포트 | HTTP/1.1 over TLS 또는 HTTP/2 over TLS는 TCP 443 |
| HTTP/3 사용 시 | QUIC 기반 UDP 443 또는 서버가 광고한 UDP 포트 |

HTTPS 접속이라고 해서 항상 TCP만 사용한다고 단정하면 안 된다.  
HTTP/1.1과 HTTP/2의 HTTPS 통신은 보통 TCP 위의 TLS를 사용하지만, HTTP/3은 QUIC 위에서 동작하고 QUIC은 UDP 기반이다.

---

## 6.2 2단계: DNS 조회

브라우저 또는 운영체제는 `www.example.com`의 IP 주소를 알아내기 위해 DNS 조회를 수행한다.

```text
www.example.com → A record / AAAA record 조회
```

| 레코드 | 의미 |
|---|---|
| A | IPv4 주소 |
| AAAA | IPv6 주소 |

DNS는 Application 계층 프로토콜이다.  
다만 실제 전송에는 일반적으로 UDP 53 또는 TCP 53을 사용할 수 있다.

운영 환경에서는 DoT(DNS over TLS), DoH(DNS over HTTPS) 같은 방식으로 DNS 질의를 수행할 수도 있다.  
따라서 “DNS는 무조건 UDP 53만 사용한다”고 이해하면 안 된다.

---

## 6.3 3단계: 라우팅 결정

IP 주소를 얻은 뒤 운영체제는 목적지 IP가 어디에 있는지 판단한다.

```text
목적지 IP가 내 서브넷 안에 있는가?
    ├─ 예: 목적지 호스트로 직접 전달
    └─ 아니오: 기본 게이트웨이로 전달
```

이 단계는 이후 `IP Address`와 `Subnet` 학습으로 연결된다.

확인해야 할 핵심은 다음과 같다.

```text
내 IP 주소
서브넷 마스크 또는 prefix length
기본 게이트웨이
라우팅 테이블
```

---

## 6.4 4단계: 다음 홉의 MAC 주소 확인

IP 패킷을 보내려면 현재 링크에서 사용할 목적지 MAC 주소가 필요하다.

IPv4에서는 일반적으로 ARP를 사용한다.

```text
기본 게이트웨이 IP 주소 → 기본 게이트웨이 MAC 주소
```

IPv6에서는 ARP가 아니라 Neighbor Discovery를 사용한다.

```text
Neighbor Solicitation / Neighbor Advertisement
```

이 단계는 이후 `ARP`와 `ICMPv6 / NDP` 학습으로 연결된다.

---

## 6.5 5단계: Transport 연결 또는 전송

HTTPS over TCP의 일반적인 흐름은 다음과 같다.

```text
Client Ephemeral Port → Server TCP 443
TCP 3-way handshake
TLS handshake
HTTP request
HTTP response
```

HTTP/3을 사용하는 경우에는 TCP가 아니라 QUIC이 사용될 수 있다.  
QUIC은 UDP 위에서 동작하며, HTTP/3은 HTTP semantics를 QUIC 위에 매핑한다.

HTTP/3의 일반적인 흐름은 다음과 같이 볼 수 있다.

```text
Client UDP Port → Server UDP 443
QUIC handshake
TLS 1.3 기반 보안 설정
HTTP/3 request
HTTP/3 response
```

> 환경 의존 사항: 실제 브라우저가 HTTP/2 over TCP를 사용할지, HTTP/3 over QUIC을 사용할지는 브라우저, 서버, 네트워크, 정책, 캐시 상태에 따라 달라질 수 있다.

---

## 6.6 6단계: Encapsulation

브라우저가 만든 HTTP 데이터는 아래로 내려가며 헤더가 붙는다.

TCP 기반 HTTPS의 예시는 다음과 같다.

```text
[HTTP Data]
    ↓ TCP Header 추가
[TCP Segment]
    ↓ IP Header 추가
[IP Packet]
    ↓ Ethernet Header/Trailer 추가
[Ethernet Frame]
    ↓ 전기/광/무선 신호로 전송
[Bits / Signal]
```

UDP 기반 HTTP/3의 예시는 다음과 같다.

```text
[HTTP/3 Data]
    ↓ QUIC Header / 보호 처리
[QUIC Packet]
    ↓ UDP Header 추가
[UDP Datagram]
    ↓ IP Header 추가
[IP Packet]
    ↓ Ethernet Header/Trailer 추가
[Ethernet Frame]
    ↓ 전기/광/무선 신호로 전송
[Bits / Signal]
```

---

## 6.7 7단계: 네트워크 장비를 통한 전달

통신 중 장비들은 계층별로 다른 정보를 본다.

| 장비 | 주로 보는 정보 | 설명 |
|---|---|---|
| 스위치 | MAC 주소 | 같은 LAN/VLAN 안에서 프레임 전달 |
| 라우터 | IP 주소 | 목적지 IP를 보고 다음 홉으로 패킷 전달 |
| 방화벽 | IP, Port, Protocol, Session, Application 정보 | 정책에 따라 허용/차단/검사 |

라우터를 지날 때 중요한 점은 다음과 같다.

```text
IP 패킷의 목적지 IP는 보통 최종 목적지를 유지한다.
하지만 TTL 또는 Hop Limit 등 IP 헤더 일부는 변경될 수 있다.
각 구간의 Layer 2 Frame은 다음 홉에 맞게 새로 만들어진다.
```

즉, 라우터는 들어온 프레임에서 IP 패킷을 꺼내고, 다음 인터페이스로 나갈 때 새 프레임에 다시 담는다.

---

## 6.8 8단계: 서버의 Decapsulation

서버는 받은 데이터를 위 계층으로 올리며 헤더를 해석한다.

TCP 기반 HTTPS의 예시는 다음과 같다.

```text
Ethernet Frame 수신
    ↓
IP Packet 추출
    ↓
TCP Segment 추출
    ↓
TLS 복호화
    ↓
HTTP Request 처리
```

HTTP/3 기반 HTTPS의 예시는 다음과 같다.

```text
Ethernet Frame 수신
    ↓
IP Packet 추출
    ↓
UDP Datagram 추출
    ↓
QUIC Packet 처리
    ↓
HTTP/3 Request 처리
```

서버 애플리케이션은 요청을 처리한 뒤 응답을 다시 같은 방식으로 캡슐화하여 클라이언트로 보낸다.

---

## 7. Encapsulation / Decapsulation과 TCP/IP 모델

## 7.1 Encapsulation

Encapsulation은 상위 계층의 데이터를 하위 계층이 자신의 헤더 또는 트레일러로 감싸는 과정이다.

```text
Application Layer
    Data

Transport Layer
    TCP/UDP Header + Data

Internet Layer
    IP Header + TCP/UDP Header + Data

Network Access Layer
    Frame Header + IP Header + TCP/UDP Header + Data + Frame Trailer
```

계층별로 보면 다음과 같다.

| 단계 | 동작 |
|---|---|
| Application | HTTP, DNS 등 애플리케이션 데이터 생성 |
| Transport | TCP 또는 UDP 헤더 추가 |
| Internet | IP 헤더 추가 |
| Network Access | Ethernet 또는 Wi-Fi 프레임 헤더/트레일러 추가 |

---

## 7.2 Decapsulation

Decapsulation은 수신 측에서 하위 계층의 헤더를 제거하고 상위 계층으로 데이터를 넘기는 과정이다.

```text
Frame 수신
    ↓
IP Packet 추출
    ↓
TCP Segment 또는 UDP Datagram 추출
    ↓
Application Data 추출
```

---

## 7.3 라우터에서의 캡슐화 변화

라우터는 일반적으로 다음과 같이 동작한다.

```text
수신 인터페이스에서 Frame 수신
    ↓
Frame 헤더 제거
    ↓
IP 헤더 확인
    ↓
TTL 또는 Hop Limit 감소
    ↓
라우팅 테이블 기반 다음 홉 결정
    ↓
송신 인터페이스에 맞는 새 Frame 생성
```

즉, 라우터를 지날 때마다 Layer 2 Frame은 바뀔 수 있지만, IP 패킷은 목적지까지 전달되기 위한 핵심 정보를 유지한다.

다만 IPv4에서는 TTL이 감소하고 IPv4 Header Checksum이 다시 계산된다.  
IPv6에서는 Hop Limit이 감소한다.  
또한 NAT, 터널링, 프록시, 보안 장비가 개입하는 경우 IP 주소, 포트, 페이로드 처리 방식이 달라질 수 있다.

---

## 8. 프레임 / 패킷 / 세그먼트 관점의 데이터 단위

| TCP/IP 계층 | 데이터 단위 | 대표 이름 | 포함되는 주요 정보 |
|---|---|---|---|
| Application | Data, Message | HTTP Message, DNS Message | 요청/응답 데이터, 도메인 이름, HTTP 헤더 등 |
| Transport | Segment / Datagram | TCP Segment, UDP Datagram | Source Port, Destination Port |
| Transport 또는 Application 인접 | QUIC Packet | QUIC Packet | Connection ID, Packet Number, Stream Frame 등 |
| Internet | Packet / Datagram | IP Packet, IP Datagram | Source IP, Destination IP, TTL 또는 Hop Limit |
| Network Access | Frame | Ethernet Frame, Wi-Fi Frame | Source MAC, Destination MAC, FCS 등 |
| Physical 성격 | Bits / Signal | Bit stream, Signal | 전기, 광, 무선 신호 |

용어 사용 시 주의할 점은 다음과 같다.

| 용어 | 주의점 |
|---|---|
| Segment | 보통 TCP 데이터 단위를 말할 때 사용 |
| Datagram | UDP 또는 IP에서 자주 사용 |
| Packet | 일반적으로 네트워크 계층 단위를 말하지만 문맥에 따라 넓게 사용될 수 있음 |
| Frame | Data Link 또는 Network Access 계층의 전송 단위 |
| QUIC Packet | UDP 위에서 동작하는 QUIC이 자체적으로 사용하는 패킷 단위 |

실무에서는 “패킷”이라는 말을 넓게 쓰는 경우가 많다.  
정확하게 설명해야 할 때는 다음처럼 구분하는 것이 좋다.

```text
TCP Segment
UDP Datagram
QUIC Packet
IP Packet / IP Datagram
Ethernet Frame
```

---

## 9. 네트워크 장비 관점에서 보는 TCP/IP 모델

## 9.1 스위치

스위치는 주로 Network Access 계층과 관련된다.

| 항목 | 설명 |
|---|---|
| 주로 보는 정보 | MAC 주소 |
| 주요 기능 | 프레임 전달, MAC Address Table 학습 |
| 대표 장애 | VLAN 불일치, 포트 down, MAC 학습 문제, 루프 |
| 관련 명령어 예 | `show mac address-table`, `show interfaces status` |

스위치는 프레임의 Source MAC 주소를 보고 MAC Address Table을 학습한다.  
Destination MAC 주소를 알고 있으면 해당 포트로 전달하고, 모르면 같은 VLAN 내에서 플러딩할 수 있다.

> 주의: L3 스위치는 라우팅 기능도 수행할 수 있으므로, “스위치 = 무조건 2계층만”이라고 단정하면 안 된다.

---

## 9.2 라우터

라우터는 주로 Internet 계층과 관련된다.

| 항목 | 설명 |
|---|---|
| 주로 보는 정보 | Destination IP Address |
| 주요 기능 | 라우팅 테이블 기반 다음 홉 결정 |
| 대표 장애 | 잘못된 기본 게이트웨이, 라우팅 누락, ACL 차단, MTU 문제 |
| 관련 명령어 예 | `show ip route`, `ip route`, `traceroute` |

라우터는 IP 패킷의 목적지 IP 주소를 보고 다음 홉을 결정한다.  
그리고 나가는 인터페이스에 맞는 새로운 Layer 2 Frame으로 다시 캡슐화한다.

라우터를 지날 때 목적지 IP 주소는 일반적으로 최종 목적지를 유지한다.  
하지만 IPv4 TTL 또는 IPv6 Hop Limit은 감소한다.  
IPv4에서는 TTL 변경으로 인해 IPv4 Header Checksum이 다시 계산된다.

---

## 9.3 방화벽

방화벽은 제품과 설정에 따라 여러 계층에 걸쳐 동작할 수 있다.

| 방화벽 유형 | 주로 보는 정보 | 관련 계층 |
|---|---|---|
| Packet Filtering | Source/Destination IP, Protocol, Port | Internet / Transport |
| Stateful Firewall | 세션 상태, TCP 상태, UDP/ICMP 흐름 상태 | L3/L4 중심 |
| Application Firewall / WAF | HTTP Host, URL, Method, Header 등 | Application |
| NGFW | 애플리케이션 식별, 사용자, 콘텐츠 검사 등 | 여러 계층 |

방화벽을 특정 계층 하나에만 고정해서 이해하면 실무에서 오해가 생길 수 있다.

예를 들어 다음 두 정책은 서로 다른 수준의 검사이다.

```text
TCP 443 포트를 허용한다.
```

```text
HTTPS 안의 특정 URL 또는 HTTP Host를 기준으로 차단한다.
```

첫 번째는 Transport 계층 중심의 정책이고, 두 번째는 Application 계층 정보가 필요하다.

또한 HTTP/3을 사용하는 환경에서는 TCP 443만 허용하고 UDP 443을 차단하면 HTTP/3 연결이 실패하거나, 클라이언트가 HTTP/2 over TCP로 fallback할 수 있다.  
다만 fallback 동작은 브라우저, 애플리케이션, 서버 설정, 네트워크 정책에 따라 달라질 수 있다.

> 환경 의존 사항: 실제 방화벽이 어느 계층까지 검사하는지는 제품, 라이선스, 모드, 암호화 복호화 설정, 정책 구성에 따라 달라진다.

---

## 10. 계층별 장애 확인 포인트

## 10.1 Network Access 계층

| 확인 항목 | 설명 |
|---|---|
| 링크 상태 | 케이블, 포트, Wi-Fi 연결, NIC 상태 확인 |
| VLAN | 올바른 VLAN에 연결되어 있는지 확인 |
| MAC 주소 | MAC Address Table 학습 여부 확인 |
| ARP / NDP | 다음 홉의 MAC 주소를 알 수 있는지 확인 |
| MTU | 프레임 크기 또는 경로 MTU 문제 확인 |

Linux 예시:

```bash
ip link
ip addr
ip neigh
sudo tcpdump -ni <interface> arp
```

Windows 예시:

```powershell
ipconfig /all
arp -a
```

스위치 예시:

```text
show interfaces status
show mac address-table
show vlan brief
```

---

## 10.2 Internet 계층

| 확인 항목 | 설명 |
|---|---|
| IP 주소 | 올바른 IP 주소가 설정되어 있는지 확인 |
| Subnet | Prefix 또는 Subnet Mask가 올바른지 확인 |
| Gateway | 기본 게이트웨이가 올바른지 확인 |
| Routing Table | 목적지로 가는 경로가 있는지 확인 |
| ICMP | ping, traceroute로 도달성 확인 |
| TTL / Hop Limit | 경로 중간에서 패킷이 폐기되는지 확인 |

Linux 예시:

```bash
ip addr
ip route
ip route get <destination-ip>
ping -c 4 <destination-ip>
traceroute <destination-host>
```

Windows 예시:

```powershell
ipconfig /all
route print
ping <destination-ip>
tracert <destination-host>
Test-NetConnection <destination-host> -TraceRoute
```

> 주의: `ping` 성공은 대상 호스트의 IP 도달성을 보여줄 수 있지만, 웹 서비스가 정상이라는 뜻은 아니다.  
> 반대로 `ping` 실패도 반드시 서버가 죽었다는 의미는 아니다. ICMP가 방화벽에서 차단될 수 있다.

---

## 10.3 Transport 계층

| 확인 항목 | 설명 |
|---|---|
| 포트 오픈 여부 | 서버가 해당 TCP/UDP 포트를 열고 있는지 확인 |
| 방화벽 정책 | Source/Destination IP, Port, Protocol 허용 여부 확인 |
| TCP 상태 | SYN 전송, SYN-ACK 수신, RST 여부 확인 |
| UDP 응답 | UDP는 연결이 없으므로 애플리케이션 응답 또는 패킷 캡처로 확인 필요 |
| QUIC 가능 여부 | UDP 443 허용 여부, HTTP/3 지원 여부 확인 |

Linux 예시:

```bash
ss -lntup
ss -ant
nc -vz <host> 443
curl -v https://<host>/
sudo tcpdump -ni <interface> tcp port 443
sudo tcpdump -ni <interface> udp port 443
```

Windows 예시:

```powershell
Test-NetConnection <host> -Port 443 -InformationLevel Detailed
netstat -ano
```

> 참고: `Test-NetConnection -Port`는 TCP 연결 확인에 주로 사용된다.  
> UDP 기반 QUIC 확인은 패킷 캡처, 브라우저 개발자 도구, 서버 로그, 전용 클라이언트 도구 등을 함께 봐야 할 수 있다.

---

## 10.4 Application 계층

| 확인 항목 | 설명 |
|---|---|
| DNS | 도메인 이름이 올바른 IP로 해석되는지 확인 |
| TLS | 인증서, SNI, TLS 버전, 암호군 문제 확인 |
| HTTP | 상태 코드, Host 헤더, URL, 프록시 문제 확인 |
| HTTP/3 | QUIC 지원 여부, Alt-Svc 광고 여부, UDP 차단 여부 확인 |
| 인증/권한 | 로그인, 토큰, 쿠키, 세션 문제 확인 |
| 애플리케이션 로그 | 서버 내부 오류 확인 |

Linux 예시:

```bash
dig www.example.com A
dig www.example.com AAAA
curl -vI https://www.example.com/
curl --resolve www.example.com:443:203.0.113.10 https://www.example.com/
```

Windows 예시:

```powershell
nslookup www.example.com
Resolve-DnsName www.example.com
Test-NetConnection www.example.com -Port 443
```

---

## 11. 장애 분석 순서 예시

웹사이트 접속 장애가 발생했다고 가정한다.

```text
증상: https://www.example.com 접속 불가
```

다음 순서로 확인한다.

---

## 11.1 내 네트워크 설정 확인

Linux:

```bash
ip addr
ip route
```

Windows:

```powershell
ipconfig /all
route print
```

확인할 것:

```text
IP 주소가 있는가?
Subnet이 올바른가?
Default Gateway가 있는가?
DNS 서버가 설정되어 있는가?
```

---

## 11.2 게이트웨이 도달 확인

Linux / Windows:

```bash
ping <default-gateway-ip>
```

게이트웨이에 도달하지 못하면 Application 계층을 보기 전에 Network Access 또는 Internet 계층 문제를 먼저 의심한다.

확인할 것:

```text
내 NIC가 up 상태인가?
VLAN이 맞는가?
ARP 또는 NDP가 정상인가?
기본 게이트웨이가 응답하는가?
```

---

## 11.3 외부 IP 도달 확인

Linux:

```bash
ping 8.8.8.8
traceroute 8.8.8.8
```

Windows:

```powershell
ping 8.8.8.8
tracert 8.8.8.8
```

외부 IP는 되지만 도메인이 안 되면 DNS 문제일 가능성이 높다.  
반대로 외부 IP도 안 되면 라우팅, 게이트웨이, 방화벽, ISP, NAT, 보안 장비 문제를 의심할 수 있다.

---

## 11.4 DNS 확인

Linux:

```bash
dig www.example.com
dig www.example.com A
dig www.example.com AAAA
```

Windows:

```powershell
nslookup www.example.com
Resolve-DnsName www.example.com
```

확인할 것:

```text
응답이 오는가?
A 또는 AAAA 레코드가 있는가?
예상한 IP와 일치하는가?
특정 DNS 서버에서만 실패하는가?
DoH 또는 DoT 환경인지 확인이 필요한가?
```

---

## 11.5 TCP 포트 확인

Linux:

```bash
nc -vz www.example.com 443
curl -vI https://www.example.com/
```

Windows:

```powershell
Test-NetConnection www.example.com -Port 443 -InformationLevel Detailed
```

확인할 것:

```text
TCP 연결이 되는가?
SYN timeout인가?
Connection refused인가?
TLS handshake에서 실패하는가?
HTTP status code는 무엇인가?
```

---

## 11.6 UDP / QUIC 확인

HTTP/3 환경에서는 UDP 443 통신 여부도 확인해야 할 수 있다.

Linux 예시:

```bash
sudo tcpdump -ni <interface> udp port 443
curl -vI --http3 https://www.example.com/
```

확인할 것:

```text
UDP 443 패킷이 나가는가?
서버에서 UDP 응답이 돌아오는가?
방화벽이 UDP 443을 차단하는가?
HTTP/3 실패 후 HTTP/2 over TCP로 fallback되는가?
```

> 참고: `curl --http3` 지원 여부는 curl 빌드 옵션과 설치된 TLS/QUIC 라이브러리에 따라 달라질 수 있다.

---

## 11.7 패킷 캡처

Linux 예시:

```bash
sudo tcpdump -ni <interface> host <server-ip>
sudo tcpdump -ni <interface> tcp port 443
sudo tcpdump -ni <interface> udp port 443
sudo tcpdump -ni <interface> arp
```

패킷 캡처에서 확인할 것:

```text
ARP 요청에 응답이 있는가?
TCP SYN이 나가는가?
SYN-ACK이 돌아오는가?
RST가 오는가?
UDP 443 응답이 돌아오는가?
TLS ClientHello 이후 응답이 있는가?
```

---

## 12. 명령어 요약표

| 목적 | Linux | Windows |
|---|---|---|
| IP 설정 확인 | `ip addr` | `ipconfig /all` |
| 링크 상태 확인 | `ip link` | `Get-NetAdapter` |
| 라우팅 확인 | `ip route` | `route print` |
| 특정 목적지 경로 확인 | `ip route get <ip>` | `Test-NetConnection <host> -TraceRoute` |
| ARP/NDP 확인 | `ip neigh` | `arp -a` |
| ICMP 도달 확인 | `ping <host>` | `ping <host>` |
| 경로 추적 | `traceroute <host>` | `tracert <host>` |
| DNS 조회 | `dig <name>` | `nslookup <name>`, `Resolve-DnsName <name>` |
| TCP 포트 확인 | `ss -lntup`, `nc -vz <host> <port>` | `netstat -ano`, `Test-NetConnection <host> -Port <port>` |
| HTTP/TLS 확인 | `curl -vI https://<host>` | `curl.exe -vI https://<host>` |
| 패킷 캡처 | `tcpdump` | Wireshark, pktmon 등 |

> 명령어 옵션은 OS, 배포판, 버전, 권한에 따라 다를 수 있다.  
> 운영 환경에서는 공식 문서 또는 `man`, `--help`, 벤더 문서를 함께 확인해야 한다.

---

## 13. 이후 학습 주제와의 연결

이 문서는 다음 주제로 자연스럽게 이어진다.

| 다음 주제 | TCP/IP 모델과의 연결 |
|---|---|
| Encapsulation | 각 계층이 데이터를 어떻게 감싸는지 설명 |
| IP Address | Internet 계층에서 호스트와 네트워크를 식별 |
| Subnet | 목적지가 로컬인지 원격인지 판단하는 기준 |
| ARP | IPv4에서 같은 링크의 다음 홉 MAC 주소를 찾는 과정 |
| ICMP | IP 계층의 오류 보고와 진단 |
| ICMPv6 / NDP | IPv6에서 이웃 탐색, 주소 해석, 라우터 탐색을 처리 |
| DNS | Application 계층에서 이름을 IP 주소로 변환 |
| Routing | Internet 계층에서 다음 홉을 선택 |
| NAT | IP 주소와 포트 변환으로 통신 경로에 개입 |
| Firewall | 계층별 정책으로 트래픽을 허용하거나 차단 |

---

## 14. 정리

TCP/IP 모델은 실제 인터넷 통신을 이해하기 위한 핵심 구조이다.

```text
Application: 무엇을 주고받을 것인가?
Transport: 어떤 프로세스끼리 통신할 것인가?
Internet: 어떤 IP 주소로 보낼 것인가?
Network Access: 다음 홉까지 어떤 프레임으로 보낼 것인가?
```

OSI 7계층은 통신 기능을 세밀하게 설명하는 참조 모델로 유용하다.  
TCP/IP 모델은 실제 인터넷에서 데이터가 이동하는 방식을 이해하는 데 더 직접적이다.

따라서 실무에서는 다음 흐름을 기준으로 생각하는 것이 좋다.

```text
DNS로 IP를 찾는다.
라우팅으로 다음 홉을 결정한다.
ARP 또는 NDP로 다음 홉의 MAC 주소를 찾는다.
TCP, UDP, QUIC으로 애플리케이션 간 통신을 수행한다.
HTTP, TLS, DNS 같은 Application 계층 프로토콜과 기술이 실제 서비스를 제공한다.
```

또한 HTTPS 통신을 볼 때는 다음을 구분해야 한다.

```text
HTTP/1.1 over TLS 또는 HTTP/2 over TLS = 일반적으로 TCP 443
HTTP/3 over QUIC = 일반적으로 UDP 443 또는 서버가 광고한 UDP 포트
```

---

## 15. 확실하지 않거나 환경 의존적인 부분

다음 내용은 환경에 따라 달라질 수 있다.

| 항목 | 설명 |
|---|---|
| TCP/IP 계층 수 | 문헌에 따라 4계층 또는 5계층으로 설명될 수 있다. 이 문서는 4계층 모델을 사용한다. |
| HTTP/3 사용 여부 | 브라우저, 서버, 네트워크 정책, 방화벽, 캐시 상태에 따라 HTTP/2 over TCP 또는 HTTP/3 over QUIC 사용 여부가 달라질 수 있다. |
| HTTPS 포트 | 일반적으로 TCP 443 또는 UDP 443을 사용하지만, 서버 광고와 설정에 따라 다른 포트가 사용될 수 있다. |
| 방화벽 동작 계층 | 제품, 라이선스, 설정, TLS 복호화 여부에 따라 L3/L4 중심일 수도 있고 L7까지 검사할 수도 있다. |
| 명령어 옵션 | OS, 배포판, 버전, 권한에 따라 결과와 옵션이 달라질 수 있다. |
| 장비 명령어 | Cisco, Juniper, Arista, Linux bridge, 클라우드 네트워크 장비 등 벤더별로 명령어가 다르다. |
| QUIC 확인 방법 | 운영체제 기본 명령만으로는 UDP 기반 QUIC 연결 상태를 TCP처럼 명확히 확인하기 어렵다. 패킷 캡처, 서버 로그, 브라우저 개발자 도구, 전용 클라이언트 도구가 필요할 수 있다. |

---

## 16. 참고 자료

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
[12]: https://datatracker.ietf.org/doc/html/rfc1034 "RFC 1034 - Domain Names: Concepts and Facilities"  
[13]: https://datatracker.ietf.org/doc/html/rfc1035 "RFC 1035 - Domain Names: Implementation and Specification"  
[14]: https://datatracker.ietf.org/doc/html/rfc8446 "RFC 8446 - The Transport Layer Security Protocol Version 1.3"  
[15]: https://datatracker.ietf.org/doc/html/rfc9110 "RFC 9110 - HTTP Semantics"  
[16]: https://datatracker.ietf.org/doc/html/rfc9113 "RFC 9113 - HTTP/2"  
[17]: https://datatracker.ietf.org/doc/html/rfc9114 "RFC 9114 - HTTP/3"  
[18]: https://datatracker.ietf.org/doc/html/rfc9000 "RFC 9000 - QUIC: A UDP-Based Multiplexed and Secure Transport"  
[19]: https://datatracker.ietf.org/doc/html/rfc7858 "RFC 7858 - DNS over TLS"  
[20]: https://datatracker.ietf.org/doc/html/rfc8484 "RFC 8484 - DNS over HTTPS"  
[21]: https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml "IANA - Service Name and Transport Protocol Port Number Registry"  
[22]: https://www.cisco.com/c/en/us/support/docs/lan-switching/ethernet/7156-36.html "Cisco - How LAN Switches Work"  
[23]: https://learn.microsoft.com/windows-server/administration/windows-commands/ping "Microsoft - ping"  
[24]: https://learn.microsoft.com/powershell/module/nettcpip/test-netconnection "Microsoft - Test-NetConnection"  
[25]: https://learn.microsoft.com/windows-server/administration/windows-commands/ipconfig "Microsoft - ipconfig"  
[26]: https://learn.microsoft.com/windows-server/administration/windows-commands/route_ws2008 "Microsoft - route"  
[27]: https://learn.microsoft.com/windows-server/administration/windows-commands/arp "Microsoft - arp"  
[28]: https://learn.microsoft.com/windows-server/administration/windows-commands/netstat "Microsoft - netstat"  
[29]: https://bind9.readthedocs.io/en/latest/manpages.html#dig-dns-lookup-utility "BIND 9 - dig"  
[30]: https://man7.org/linux/man-pages/man8/ip.8.html "Linux man-pages - ip"  
[31]: https://man7.org/linux/man-pages/man8/ss.8.html "Linux man-pages - ss"  
[32]: https://man7.org/linux/man-pages/man8/ip-neighbour.8.html "Linux man-pages - ip-neighbour"  
[33]: https://manpages.ubuntu.com/manpages/jammy/man8/tcpdump.8.html "Ubuntu Manpage - tcpdump"  
[34]: https://manpages.ubuntu.com/manpages/jammy/man8/traceroute.8.html "Ubuntu Manpage - traceroute"
