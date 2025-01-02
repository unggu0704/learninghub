---
author: 김규형
date: 2023-09-11 14:10:00 +0800
categories: [NHN Academy, ETC]
tags: [CS, NHN Acedemy]
render_with_liquid: true
---

# Computer Network (23. 10. 25)

## Computer Network

## **Computer network 이란?**

- 서로 데이터를 교환하고 리소스를 공유할 수 있는 상호 연결된 Computing device의 집합
- 프로토콜을 사용하여 통신
- 다양한 형태로 구성(Topology)

### Node

- Computer device간 메세지 전달을 위해 중간 역할 수행하는 디바이스들
    - Switch
        - OSI 7 계층의 data link 층에서 작동
        - 두 network를 연결해 하나의 network로 동작
    - Firewall
        - 네트워크의 보안 및 엑세스 규칙 제어위한 SW
        - 허용되지 않은 접근은 원칙적 차단
    - Modem
        - 아날로그 to 디지털
    - Router
        - OSI 7계층의 network층에서 작동
        - Packet 전달에 이용
        - Routing의 경로 결정
    - 등등등…

### Protocol

- 네트워크를 통해 정보 교환을 위한 일련의 규칙
- Protocol 종류
    - IEEE 802.11 (Wi-fi 프로토콜)
    - HTTP (IP / TCP 프로토콜)

### OSI 7계층

- Physical Layer: 이 계층은 실제 데이터 전송을 다룹니다(전선, 광섬유 등). 하드웨어 특성, 신호 및 전송 속도를 정의한다.
- Data Link Layer계층: 이 계층은 직접 연결된 두 노드 간의 신뢰할 수 있는 링크를 만듭니다. 프레임 형식, 오류 감지 및 흐름 제어와 같은 문제를 처리한다. Ethernet은 데이터 링크 계층 기술의 예입니다.
- Network Layer: 네트워크 계층은 다중 연결된 네트워크를 통해 패킷을 출발지에서 목적지로 라우팅한다. IP(Internet Protocol)가 이 계층에서 작동한다.
- Transport Layer: 이 계층은 발신자와 수신자 간의 종단 간 통신을 보장한다. 데이터 분할, 흐름 제어 및 오류 정정을 관리한다. TCP(Transmission Control Protocol) 및 UDP(User Datagram Protocol)는 전송 계층 프로토콜입니다.
- Session Layer: 이 계층은 통신 세션을 설정, 관리 및 종료한다. 세션 동기화 및 대화 제어를 처리한다.
- Presentation Layer: 데이터 번역, 압축 및 암호화를 담당한다. 데이터가 발신자와 수신자 양쪽에서 읽을 수 있는 형식으로 제공됨을 보장한다.
- Application Layer: 가장 상위 계층으로 응용 프로그램별 프로토콜 및 데이터 형식을 처리한다. 웹 브라우저, 이메일 클라이언트 및 파일 전송 소프트웨어와 같은 사용자 수준 응용 프로그램이 작동하는 곳입니다.

### TCP / IP 4계층

**Host-to-Network Layer**

Host-to-Network(네트워크 인터페이스라고도 함)는 가장 낮은 TCP/IP 계층로, 패킷을 수신하고 특정 네트워크를 통해 전송하는 역할을 담당한다. Network interface는 장치 드라이버 또는 자체 데이터 링크 프로토콜을 사용하는 복잡한 서브시스템으로 구성될 수 있다.

**Internet Protocol(IP) Layer**

Internet protocol layer는 한 대의 기계에서 다른 대의 기계로의 통신을 처리한다. 이 layer는 transport layer에서 데이터를 전송하도록 요청을 수락하며, 데이터를 전송할 기계의 식별 정보와 함께 받는다. 그런 다음 데이터를 IP 데이터그램으로 래핑하고 데이터그램 헤더를 채우며, 데이터그램을 전달 방법을 결정하기 위해 라우팅 알고리즘을 사용하고 데이터그램을 전송하기 위해 적절한 장치 드라이버에 전달한다.

**Transport Layer**

Transport layer의 주요 목적은 한 응용 프로그램에서 다른 응용 프로그램으로의 통신을 제공하는 것이다. 소프트웨어는 전송되는 데이터 스트림을 ISO 용어로 패킷이라고 하는 더 작은 조각으로 나누고 각 패킷과 목적지 정보를 다음 레이어로 전달한다.

**Application Layer**

Application layer는 TCP/IP 인터넷 전체에서 사용 가능한 서비스에 접근하는 사용자 호출 응용 프로그램으로 구성된다. 응용 프로그램은 데이터를 전송 레이어로 전달하기 위해 필요한 형식으로 전달한다.

### Network Issue

- 충돌 회피
    - 여러기기가 동시에 데이터 전송시의 충돌을 회피
    - CSMA/CD, CSMA/CA와 같은 방식으로 해결
- 디지털 to 아날로그
- Error Detection and Correction
    - CheckSUM , 패리티 비트 등으로 전송 중 오류를 감지
- Routing
    - OSPF, BGF, RIP 등등 경로 지정 프로토콜 사용
- Network Protocol
    - TCP/IP, UDP, FTP 등등
    - 각 프로토콜의 속도 및 신뢰성
- Security
- Qos
    - 서비스 품질은 얼마나?

## DNS

- 컴퓨터 네트워크에서 사용되는 계층화된 분산 데이터베이스 시스템이다.
- 도메인 이름 → IP 주소 / IP 주소 → 도메인 이름으로 변환