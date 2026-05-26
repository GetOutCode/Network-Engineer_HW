# Network Labs

이 폴더는 PNETLab 및 Packet Tracer 기반 네트워크 인프라 실습 결과를 정리하는 공간입니다.

단순히 명령어를 입력하고 ping 성공 여부만 확인하는 것이 아니라,  
네트워크 인프라 엔지니어 관점에서 토폴로지 설계, IP 주소 계획, 장비 설정, CLI 기반 검증, 장애 원인 분석 과정을 함께 기록합니다.

실습은 L2 Switching, L3 Routing, Network Security, Server/DMZ/Proxy 연동 흐름을 중심으로 구성합니다.


## Lab Structure

각 실습은 아래 구조로 정리합니다.

```text
Lab_Name/
├── README.md
├── configs/
├── screenshots/
└── verification/
```

| 항목 | 내용 |
|---|---|
| README.md | 실습 목적, 토폴로지, IP 주소 계획, 설정 과정, 검증 결과 |
| configs/ | 라우터, 스위치, 서버 설정 백업 |
| screenshots/ | PNETLab 토폴로지, CLI 결과, ping 테스트 화면 |
| verification/ | show 명령어 결과, 테스트 로그, 장애 분석 기록 |

---

## 01. Basic Network Connectivity

네트워크 간 기본 통신과 라우팅 동작을 이해하기 위한 실습입니다.

| Lab | Topic | Goal |
|---|---|---|
| 01_Static_Routing | Static Routing | 서로 다른 네트워크 간 정적 라우팅 구성 |
| 02_Default_Route | Default Route | 외부망 방향 기본 경로 설정 |
| 03_Routing_Verification | Routing Table | 라우팅 테이블 기반 경로 확인 |

### Key Verification Commands

```text
show ip interface brief
show ip route
ping
traceroute
show arp
```

---

## 02. L2 Switching Labs

스위치 기반 네트워크 분리와 VLAN 전달 구조를 이해하기 위한 실습입니다.

| Lab | Topic | Goal |
|---|---|---|
| 04_VLAN_Access_Port | VLAN / Access Port | VLAN을 이용한 네트워크 분리 |
| 05_Trunk_8021Q | Trunk / 802.1Q | 스위치 간 여러 VLAN 전달 |
| 06_STP_Basic | STP | L2 Loop 방지 및 Root Bridge 확인 |

### Key Verification Commands

```text
show vlan brief
show interfaces trunk
show spanning-tree
show mac address-table
```

---

## 03. L3 Routing Labs

VLAN 간 통신과 동적 라우팅을 구성하고 경로 학습 과정을 확인하는 실습입니다.

| Lab | Topic | Goal |
|---|---|---|
| 07_InterVLAN_Routing | Inter-VLAN Routing | VLAN 간 Gateway 구성 |
| 08_OSPF_Basic | OSPF | 동적 라우팅 구성 |
| 09_OSPF_Failure_Test | OSPF Failure Test | 링크 장애 시 경로 변화 확인 |

### Key Verification Commands

```text
show ip route
show ip ospf neighbor
show ip protocols
traceroute
```

---

## 04. Network Security Labs

네트워크 접근 제어와 주소 변환 구조를 이해하기 위한 실습입니다.

| Lab | Topic | Goal |
|---|---|---|
| 10_ACL_Basic | ACL | 특정 대역 또는 서비스 접근 제어 |
| 11_NAT_PAT | NAT / PAT | 내부망에서 외부망 접근 구성 |
| 12_Access_Control_Flow | Access Control Flow | 허용/차단 정책 흐름 검증 |

### Key Verification Commands

```text
show access-lists
show ip nat translations
show ip nat statistics
ping
telnet
curl
```

---

## 05. Server / DMZ / Proxy Labs

서버, DMZ, Proxy가 포함된 네트워크 흐름을 이해하기 위한 실습입니다.

| Lab | Topic | Goal |
|---|---|---|
| 13_Linux_Server_Integration | Linux Server | 서버 IP, Gateway, Routing 확인 |
| 14_DMZ_Basic | DMZ | 내부망, DMZ, 외부망 분리 |
| 15_DMZ_Proxy_Update_Flow | DMZ Proxy | 내부 서버가 DMZ Proxy를 통해 외부 서버에 접근하는 흐름 구현 |

### DMZ Proxy Flow

```text
Internal Server
→ Firewall / Router
→ DMZ Proxy
→ External Server
```

### Example Policy

| Source | Destination | Service | Action |
|---|---|---|---|
| Internal Server | DMZ Proxy | TCP 3128 | Permit |
| Internal Server | External Server | Any | Deny |
| DMZ Proxy | External Server | HTTP / HTTPS | Permit |
| External Server | Internal Server | Any | Deny |

---

## Verification Standard

모든 실습은 가까운 구간부터 검증합니다.

```text
1. PC 또는 Server → Gateway ping
2. Router ↔ Router ping
3. End-to-End ping
4. show ip interface brief 확인
5. show ip route 확인
6. show arp 확인
7. show mac address-table 확인
8. ACL, NAT, 방화벽 정책 확인
9. 장애 발생 시 원인별 점검 기록
```

---

## Troubleshooting Checklist

통신 실패 시 아래 순서로 점검합니다.

```text
1. 장비가 정상적으로 부팅되어 있는가?
2. 인터페이스가 up/up 상태인가?
3. IP Address와 Subnet Mask가 올바른가?
4. PC 또는 Server의 Gateway가 올바른가?
5. VLAN Access Port 설정이 올바른가?
6. Trunk에서 VLAN이 허용되어 있는가?
7. Routing Table에 목적지 경로가 있는가?
8. ACL 또는 방화벽 정책에 의해 차단되고 있지 않은가?
9. NAT 변환이 정상적으로 발생하는가?
10. ARP 또는 MAC Address Table이 정상적으로 학습되었는가?
```

---

## Portfolio Goal

이 실습 저장소의 목표는 다음 역량을 증명하는 것입니다.

- PNETLab에서 네트워크 토폴로지를 직접 구성할 수 있다.
- L2 Switching과 L3 Routing 구조를 이해하고 설정할 수 있다.
- VLAN, Trunk, STP, Static Routing, OSPF를 CLI로 구성하고 검증할 수 있다.
- ACL, NAT를 이용해 네트워크 접근 제어와 주소 변환을 구성할 수 있다.
- 서버, DMZ, Proxy가 포함된 네트워크 흐름을 설명할 수 있다.
- 장애 발생 시 인터페이스, IP, Gateway, Routing, VLAN, ACL, NAT 순서로 원인을 추적할 수 있다.

---

## Lab Progress

| No | Lab | Status |
|---|---|---|
| 01 | Static Routing | Planned |
| 02 | Default Route | Planned |
| 03 | Routing Verification | Planned |
| 04 | VLAN Access Port | Planned |
| 05 | Trunk 802.1Q | Planned |
| 06 | STP Basic | Planned |
| 07 | Inter-VLAN Routing | Planned |
| 08 | OSPF Basic | Planned |
| 09 | OSPF Failure Test | Planned |
| 10 | ACL Basic | Planned |
| 11 | NAT PAT | Planned |
| 12 | Access Control Flow | Planned |
| 13 | Linux Server Integration | Planned |
| 14 | DMZ Basic | Planned |
| 15 | DMZ Proxy Update Flow | Planned |
