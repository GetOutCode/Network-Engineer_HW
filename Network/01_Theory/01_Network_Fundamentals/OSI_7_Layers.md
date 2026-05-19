# OSI 7계층 정리

## 1. 개요

OSI 7계층은 네트워크 통신 과정을 계층별로 나누어 이해하기 위한 모델이다.  
장애 원인을 계층별로 분리해서 확인하는 기준으로 활용된다.

예를 들어 통신이 되지 않을 때 물리 연결 문제인지, VLAN 문제인지, IP 설정 문제인지, 라우팅 문제인지, 방화벽 정책 문제인지 구분하기 위해 OSI 계층 관점으로 접근할 수 있다.

## 2. OSI 7계층 구조

| 계층 | 이름 | 주요 역할 | 대표 장비/프로토콜 |
|---|---|---|---|
| 7계층 | Application | 사용자 서비스 제공 | HTTP, HTTPS, DNS, FTP, SMTP |
| 6계층 | Presentation | 데이터 표현, 암호화, 압축 | TLS, SSL, Encoding |
| 5계층 | Session | 세션 연결 및 유지 | Session Control |
| 4계층 | Transport | 포트 기반 통신, 신뢰성 제어 | TCP, UDP |
| 3계층 | Network | IP 주소 기반 통신, 라우팅 | IP, ICMP, Router, L3 Switch |
| 2계층 | Data Link | MAC 주소 기반 통신 | Ethernet, Switch, VLAN, STP |
| 1계층 | Physical | 전기적 신호, 케이블, 물리 연결 | Cable, Hub, NIC |

## 3. 계층별 핵심 이해

### 1계층 Physical Layer

1계층은 케이블, 전기 신호, 포트 연결 상태처럼 물리적인 통신 환경을 담당한다.  
장애 확인 시 케이블 연결, 포트 Link 상태, 장비 전원, 인터페이스 상태를 먼저 확인한다.

확인 예시:

```bash
show interfaces status
show interfaces
