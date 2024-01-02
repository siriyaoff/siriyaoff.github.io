---
layout: single
title: "네트워크"
categories:
  - Dev
tags:
  - Network
  - Throughput
  - Bandwidth
  - Latency
  - Network topology
  - Tree
  - Bus
  - Star
  - Ring
  - Mesh
  - TCP/IP
  - OSI
  - 3-way Handshake
  - TCB
  - SYN
  - ACK
  - 4-way Handshake
  - FIN
  - TIME_WAIT
  - UDP
  - Datagram
  - Ethernet frame
  - Bit
  - Frame
  - Packet
  - Segment
  - Message
  - Loadbalancer
  - Switch
  - Router
  - ARP
  - Hop by hop
  - Gateway
  - DHCP
  - NAT
  - HTTP
  - HTTPS
  - SEO
---

# 2.1 네트워크의 기초

- 네트워크 : node와 link가 연결되어 리소스를 공유하는 집합

## 2.1.1 처리량과 지연 시간

- 처리량(Throughput) : 링크를 통해 전달되는 단위 시간당 데이터양(bps)
- 대역폭(Bandwidth) : 네트워크의 최대 전송 속도(Mbps)
  - 전자공학에서 대역폭은 신호가 차지하는 주파수의 범위(Hz)임
- 지연 시간(Latency) : 두 노드 사이를 왕복하여 요청이 처리되는데 걸린 시간

## 2.1.2 네트워크 토폴로지와 병목 현상

- Network topology : 노드와 링크 배치 방식에 대한 방법론
- Tree topology : 트리 형태
  - 노드 추가, 삭제가 쉬움
  - 특정 노드에 트래픽이 집중될 때 하위 노드에 영향을 끼칠 수 있음
- Bus topology : 중앙 케이블에 여러 개의 노드가 연결된 형태
  - 노드 추가, 삭제가 쉬움
  - LAN에 사용됨
  - 설치 비용이 적고 신뢰성이 우수함
  - 스푸핑이 가능함
    - 스푸핑 : 스위치의 기능을 마비시켜 특정 노드에 패킷이 오도록 처리하는 것
- Star topology : 중앙의 노드에 모든 노드가 연결된 형태
  - 노드 추가, 삭제, 에러 탐지가 쉬움
  - 패킷 충돌 발생 가능성이 적음
  - 중앙 노드에 장애가 발생하면 전체 네트워크에 문제가 발생
  - 설치 비용이 비쌈
- Ring topology : 양 옆의 두 노드끼리만 연결
  - 노드 고장을 쉽게 탐지 가능
  - 패킷 충돌 발생 가능성이 적음
  - 네트워크 구성 변경이 어려움
  - 회선에 장애가 나면 전체 네트워크에 문제가 발생
- Mesh topology : 모두 연결
  - 트래픽 분산 처리 가능
  - 노드의 추가가 어려움
  - 설치, 운용 비용이 비쌈
- 병목 현상을 찾을 때 topology를 사용함

## 2.1.3 네트워크 분류

- LAN(Local Area ~), MAN(Metropolitan ~), WAN(Wide ~)

## 2.1.4 네트워크 성능 분석 명령어

- 병목 현상의 주된 원인
  - Bandwidth
  - Topology
  - 서버 CPU, 메모리 사용량
- `ping` : 대상 노드로 패킷을 전송
  - ICMP 프로토콜을 지원하지 않는 대상으로는 불가능
- `netstat` : 접속된 서비스들의 네트워크 상태를 표시
  - 프로토콜, 로컬, 외부 주소, 포트, 상태 포함
- `nslookup` : 특정 도메인의 IP 확인
- `tracert`, `traceroute` : 특정 노드까지 네트워크 경로 확인

## 2.1.5 네트워크 프로토콜 표준화

- IEEE802.3 : 유선 랜 프로토콜

# 2.2 TCP/IP 4계층 모델

- Internet protocol suite : 프로토콜의 집합
  - TCP/IP 4계층 모델, OSI 7계층 모델로 설명함

## 2.2.1 계층 구조

- TCP/IP(Transmission Control Protocol/Internet Protocol)
  - 각 계층은 서로 영향을 받지 않도록 설계됨

![TCP-IP-OSI-7-Layer.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/TCP-IP-OSI-7-Layer.png)

### Application Layer

- 실질적인 서비스를 제공하는 계층
- FTP(장치간 파일 전송), HTTP, SSH(원격 접속용 보안), SMTP, DNS(도메인, ip 매핑)

### Transport Layer

- 송신자, 수신자를 연결하는 통신 서비스 제공(TCP, UDP, QUIC)
- TCP
  - 가상회선 패킷 교환 방식 사용
    ![virtual-circuit-packet-transportation.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/virtual-circuit-packet-transportation.png)
    - 가상회선 : 송신자, 수신자 사이에 임의로 정한 하나의 경로
    - 각 패킷에 가상회선 식별자가 포함되어 모든 패킷은 가상회선으로만 전송됨
    - 패킷 사이 순서 보장
  - 3-way Handshake
    ![3-way-handshake.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/3-way-handshake.png)
    - TCB(TCP Control Block) : connection state, sequece 번호 등의 연결에 필요한 정보를 가진 구조체
    1. client에서 `SYN`(client ISN) 보냄
       - client : `SYN_SENT`
       - server : `LISTEN`
    2. server에서 `SYN` 받고 `SYN`(server ISN)+`ACK`(client ISN +1) 보냄
       - client : `SYN_SENT`
       - server : `SYN_RECEIVED`
    3. client에서 `SYN`+`ACK` 받고 `ACK`(server ISN +1) 보냄
       - client : `ESTABLISHED`
       - server : `ACK` 받은 후 `ESTABLISHED`
  - 4-way Handshake
    ![4-way-handshake.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/4-way-handshake.png)
    1. client에서 `FIN` 보냄
       - client : `FIN_WAIT_1`
       - server : `ESTABLISHED`
    2. server에서 `FIN` 받고 앱 닫을 준비하고 `ACK` 보냄
       - client : `FIN_WAIT_1`
       - server : `CLOSE_WAIT`
    3. server에서 앱 닫을 준비가 된 후 `FIN` 보냄
       - client : `ACK` 받으면 `FIN_WAIT_2`
       - server : `LAST_ACK`
    4. client에서 `FIN` 받고 `ACK` 보냄
       - client : `TIME_WAIT`, 일정 시간(MSL) 동안 세션을 남겨놓고 잉여 패킷을 기다린 다음 `CLOSED`
       - server : `ACK` 받으면 `CLOSED`
    - `TIME_WAIT` 존재 이유
      - 지연 패킷
      - passive close의 `FIN`에 대한 `ACK`를 보내고 passive close가 닫혔는지 확인 후 종료하기 위함(`TIME_WAIT`이 없으면 `FIN`을 받자마자 `CLOSED`가 되기 때문에 `ACK`를 passive close로 보내지 못함)
      - Ubuntu는 1분, Windows는 4분
- UDP
  - 데이터그램 패킷 교환 방식 사용
    ![datagram-packet-transportation.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/datagram-packet-transportation.png)
    - 각 패킷들이 독립적으로 이동함
    - 패킷 사이 순서 보장되지 않음

### Internet Layer

- 패킷을 IP 주소로 지정된 노드까지 라우팅하는 계층
- IP, ARP, ICMP

### Network Interface Layer(Link Layer)

- 데이터를 전기 신호로 변환, 전송하고 Ethernet 등의 프로토콜을 통해 제어하는 계층
  - OSI에선 물리 계층(전기 신호 전송), 데이터 링크 계층(에러 확인, 흐름, 접근 제어 등)으로 구분함
- Ethernet
  - IEEE802.3 : 유선 LAN에서 사용하는 프로토콜, 전이중화 통신 사용
    - 전이중화 통신 : 송신, 수신로를 나눠 동시에 송수신 가능
  - CSMA/CD : 이전에 사용하던 반이중화 통신(충돌이 발생하면 일정 시간 후 재전송)
  - 이더넷 프레임
    ![ethernet-frame.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/ethernet-frame.png)
    - Preamble : 프레임의 시작을 알림(0, 1 반복)
    - SFD(Start Frame Delimiter) : 다음 바이트부터 DA가 시작됨을 알림(`10101011`)
    - DA(Destination Address) : 목적지의 MAC 주소
    - SA(Source Address) : 출발지의 MAC 주소
    - EtherType/Length : `0x600` 이하일 경우 IEEE802.3의 Length, 이상일 경우 DIX 2.0의 Type으로 해석
    - Data/Payload : 46 B 보다 작을 경우 Padding으로 채워짐
    - FCS(CRC) : DA+SA+Length+Data의 영역을 계산하여 에러 판별
      - 송신측에서 추가, 수신측에서 확인하여 에러 프레임은 버림
- 무선 랜
  - IEEE802.11
  - 송 수신이 같은 채널로 이루어지기 때문에 반이중화 통신 사용
  - CSMA/CA : 캐리어 감지, IFS(Inter FrameSpace)로 기다렸다가 무선 매체 사용
  - BSS(Basic Service Set) : 하나의 AP를 이용
  - ESS(Extended Service Set) : 하나 이상의 연결된 BSS 그룹, 다른 장소로 이동하며 네트워크 사용 가능

### 계층 간 데이터 송수신 과정

![tcp-ip-stack-packet.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/tcp-ip-stack-packet.png)

- Message : Application layer에서 생성된 PDU
- Segment : TCP(L4) header가 붙은 PDU
  - UDP일 경우 Datagram
- Packet : IP(L3) header가 붙은 PDU
- Frame : Frame header + trailer가 붙은 PDU
- Bit : 물리 계층에서 사용되는 PDU

# 2.3 네트워크 기기

## 2.3.1 네트워크 기기의 처리 범위

- Application Layer : L7 스위치
- Internet Layer : L3 스위치, 라우터
- Data Link Layer : L2 스위치, 브릿지
- Physical Layer : NIC, 리피터, AP

## 2.3.2 Application 계층을 처리하는 기기

### L7 스위치(로드밸런서)

- 목적지와 연결된 포트로만 신호를 보내는 네트워크 장비
- URL, 서버, 캐시, 쿠키 등을 기반으로 요청을 여러 서버로 나눠 부하를 분산함
- 가상 IP를 사용해 서버 이중화 가능
- 필터링, 모니터링 가능
- Health check을 통해 장애가 발생한 서버 확인
  - Health check : TCP, HTTP 등의 다양한 방법으로 요청을 보내어 요청이 정상적으로 이루어지는지 확인
- L4 스위치와의 차이
  - L4 스위치 : Tranport 계층을 처리
    - IP, port를 기반으로 트래픽 분산
    - Message를 기반으로 인식하지 못함
    - 스트리밍 관련 서비스에서는 사용할 수 없음
    - AWS에서는 NLB
  - L7 스위치 : Application 계층을 처리
    - IP, port, URL, HTTP header, 쿠키 등을 기반으로 트래픽 분산
    - AWS에서는 ALB

## 2.3.3 Internet 계층을 처리하는 기기

### 라우터(L3 스위치)

- 라우팅 : 다른 네트워크끼리 통신할 때 패킷 소모를 최소화하고 최적의 경로로 패킷을 포워딩하는 작업
- L2 스위치 + 라우팅

## 2.3.4 Data Link 계층을 처리하는 기기

### L2 스위치

- MAC 주소 테이블을 통해 장치들의 MAC 주소 관리
- 패킷 전송 담당
- 패킷의 MAC 주소를 읽어 스위칭함
  - IP 주소 기반의 라우팅은 불가능
  - 목적지가 스위치의 MAC 주소 테이블에 없다면 전체 포트에 전달

### 브릿지

- MAC 주소 테이블을 사용해 LAN 끼리의 연결 가능

## 2.3.5 Physical 계층을 처리하는 기기

- NIC(Network Interface Card) : MAC 주소를 가짐
- Repeater : 전기 신호 증폭
  - 광케이블로 인해 현재는 사용 안함
- AP(Access Point) : 유선랜과 무선랜을 연결시켜줌

# 2.4 IP 주소

## 2.4.1 ARP

- ARP(Address Resolution Protocol) : IP로부터 MAC을 구하는 프로토콜
  - RARP : MAC→IP 변환
  - A에서 ARP Request broadcast를 보냄 → 해당 IP를 가진 B에서 ARP Reply unicast를 보냄(스위치 ARP table에 B가 있을 경우 스위치에서 보냄)

## 2.4.2 Hop by hop 통신

- IP 주소를 라우팅을 통해 찾아가는 통신 방법
- 라우팅 테이블 : Destination, Gateway, Genmask, Flags, Metric, Ref, Use, Iface 포함
  - Gateway : 서로 다른 통신망, 프로토콜을 사용하는 네트워크 간의 통신을 지원해줌
    - 같은 네트워크 내에서는 보통 MAC 주소로 통신함
    - 게이트웨이는 다른 대역으로 라우팅 해줄 수 있는 L3 스위치 이상의 장비임
    - 기본 게이트웨이 : 라우터(공유기, 보통 `192.168.0.1`)
  - Genmask(Netmask) : `255.255.255.255`(목적지 host의 주소), `0.0.0.0`(기본 게이트웨이 주소)
  - Flags
    - U(up) : 경로가 살아있음
    - H(host) : 목적지가 host 주소임
    - G(Gateway) : Gateway로 향하는 경로임
  - Metric : 목적지 네트워크까지의 거리
  - Ref : 경로를 참조한 횟수
  - Use : 경로를 사용한 횟수
  - Iface : Network interface

## 2.4.3 IP 주소 체계

- IPv4 : 8 bit 단위, 32 bit
- IPv6 : 16 bit 단위, 64 bit

### 클래스 기반 할당 방식

![ip-classes.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/ip-classes.png)

- 가장 앞 주소(`1.0.0.0`) : 네트워크 구별 주소(클래스 A 네트워크 `1.0.0.0` 기준)
- 가장 뒷 주소(`1.255.255.255`) : broadcast 주소(클래스 A 네트워크 `1.0.0.0` 기준)

### DHCP

- DHCP(Dynamic Host Configuration Protocol) : 자동으로 IP 주소와 TCP/IP 기본 설정을 제공해주는 프로토콜

### NAT

- NAT(Network Address Translation) : IP 패킷의 출발지, 목적지의 IP 주소와 포트를 바꾸면서 라우터를 통해 트래픽을 주고받는 기술
  - IP 주소 절약 : 공인 IP, 사설 IP 변환
  - 보안 : 사설 IP 주소와 외부 IP 주소가 다르기 때문에 내부 호스트 보호 가능
  - 호스트 숫자에 따라 접속 속도가 느려질 수 있음

# 2.5 HTTP

## 2.5.1 HTTP/1.0

- 한 연결당 하나의 요청을 처리
- 요청마다 3-way handshake를 열어야 하기 때문에 RTT(Round Trip Time) 증가
- 해결 방안
  - 이미지 스플리팅 : 여러 이미지를 합친 하나의 이미지를 한 번에 받고 CSS의 `background-position`을 사용해서 분할하여 표시
  - 코드 압축 : 개행 문자, 빈칸 등을 없애서 최대한 용량을 줄임
  - 이미지 Base64 인코딩 : 이미지를 64진법 문자열로 인코딩
    - 이미지에 대해 HTTP 요청을 할 필요가 없어짐
    - 크기가 37% 정도 커짐

## 2.5.2 HTTP/1.1

- 한 번 TCP 초기화를 한 이후 keep-alive 옵션으로 여러 개의 파일을 송수신할 수 있음
  - 1.0에도 있지만 기본 옵션이 아니었음
  - 3-way handshake가 한 번만 일어남
  - 요청할 리소스 개수가 많으면 대기 시간이 길어짐
- HOL Blocking(Head Of Line Blocking) : 같은 큐의 첫 패킷에 의해 지연이 발생하면 다른 것들도 전송 시간이 지연됨
- 헤더가 압축되지 않아 무거움

## 2.5.3 HTTP/2

- SPDY 프로토콜에서 파생됨
- 요청의 우선순위 처리 지원
- 멀티플렉싱 : 여러 스트림을 사용하여 송수신 지원
  - 하나의 TCP 커넥션 내부에 여러 스트림이 존재하여 병렬로 여러 요청을 받을 수 있음
  - HOL Blocking을 해결함
- 헤더 압축 : HPACK 압축(Huffman code를 사용)
- 서버 푸시 : client의 요청 없이 서버에서 리소스를 바로 푸시할 수 있음(html 요청이 들어오면 포함된 css 파일 푸시 등)

## 2.5.4 HTTPS

- HTTPS : application 계층과 tranport 계층 사이에 신뢰 계층인 SSL/TLS 계층을 넣은 HTTP의 보안 버전
  - HTTP/2 또한 HTTPS 위에서 동작함
- SSL/TLS(Secure Socket Layer, Transport Layer Security Protocol)
  - 전송 계층에서 보안을 제공하는 프로토콜
  - SSL → TLS로 버전이 올라감
  - 보안 세션을 기반으로 인터셉터 방지
    - TLS Handshake를 통해 보안 세션 생성
  - Handshake를 위한 1-RTT가 실행된 후 데이터 송수신이 일어남
    - client의 cypher suite(프로토콜 + AEAD 사이퍼 모드 + 해싱 알고리즘)를 서버에서 받아 확인 후 인증 메커니즘 시작

### SEO

- HTTPS를 지원하는게 SEO를 높여줌
- `<link>` 태그에 `rel="canonical"` 프로퍼티를 설정해서 표준 URL 명시
- 사이트맵 관리

### HTTPS 구축 방법

- CA에서 구매한 인증키를 기반으로 구축
- 서버 앞단에 HTTPS를 제공하는 로드밸런서 배치
- 서버 앞단에 HTTPS를 제공하는 CDN을 배치

## 2.5.5 HTTP/3

- TCP 대신 UDP 사용
- QUIC 계층이 추가됨
  - 3-way handshake가 없음
  - 첫 연결 설정에 1-RTT 소요
  - FEC(Forward Error Correction) 적용됨(수신측에서 에러 검출, 수정)
