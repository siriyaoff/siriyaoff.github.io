---
layout: single
title: "AWS SA Associate Dump Note"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - AWS
  - SAA
  - S3
  - SQS
  - SNS
  - API Gateway
  - Lambda
  - Endpoint
  - Glacier
  - RDS
  - DynamoDB
  - Kinesis
  - Redshift
  - NAT
  - WAF
  - Shield Advanced
  - Cost Explorer
  - Config
  - CloudTrail
  - FSx
  - Direct Connect
---

## Miscellaneous

- Global Accelerator : optimal regional endpoint로의 라우팅 지원
  - Endpoint : NLB, ALB, EC2, EIP 등의 주소로 설정 가능
- EC2 spot instance : 배치작업 등에 효율적
  - 시작하려는 인스턴스의 현재 가격만 지불하면 됨
  - 온디맨드보다 7-90% 절감된 비용
  - Reserve instance : 1, 3년 등의 기간에 대해 일시 지불
  - On-demand : 사용한 만큼 지불
- 단일 실패 지점 : 시스템 구성 요소 중 동작하지 않으면 전체 시스템이 중단되는 요소
- CloudFormation : IaC 서비스
- API Gateway : RESTful
- raw data store in S3 → GLUE with ETL
- Elastic BeanStalk : 비쌈
- CF, ALB : UDP 지원 x
- Application Auto Scaling : EC2 이외의 다른 AWS 서비스에 대한 오토 스케일링 지원
- Lambda@Edge : CF가 실행하는 람다 함수
- 피크 시간대에 밀리초 단위 지연 시간으로 요청 처리하는 기본 구조
  - S3 버킷으로 정적 웹 호스팅
    - S3를 origin으로하는 cloudfront 배포
  - API Gateway, lambda로 백엔드 구성
  - DynamoDB로 저장
- efs, s3 가격 비교
- KMS, CMK
- Auto Scaling 수명 주기 hook을 사용해서 시작, 종료될 때 스크립트 실행 가능

## S3

- S3 보존 모드
  - 거버넌스 : 객체 버전 메타데이터 수정, 삭제, 잠금 설정 변경 등을 위해선 각자의 권한이 필요함(e.g. 객체 삭제, 기한 변경 : `s3:BypassGovernanceRetention` 권한 필요)
  - 규정 준수 : 권한이 있더라도 처음에 지정한 기간 동안은 수정, 삭제 불가능
- Endpoint : 인터넷 접속 없이 트래픽 이동 가능
  - S3 Gateway endpoint
    - S3 public IP 사용
    - 하나의 S3 DNS 사용
    - 온프레미스 액세스 안됨
    - 다른 리전 안됨
    - 비용 x
  - S3 Interface endpoint
    - VPC private IP 사용
    - endpoint 별로 S3 DNS 필요
    - 온프레미스 액세스 허용
    - VPC 피어링, AWS Transit Gateway를 통해 다른 리전 액세스 허용
    - 비용 o
- S3 Intelligent-Tiering : Frequent, Infrequent, Archive instant 3개의 티어로 데이터를 나눠 저장함
  - 액세스 패턴이 변경되는 데이터에 대해 스토리지 비용 절감 가능
- S3 Glacier
  - 스토리지 클래스
    - Instant Retrieval : 분기 당 한 번, 밀리초 단위 액세스
    - Flexible Retrieval : 연 1-2회, 수 분에서 몇 시간 내 액세스
    - Deep Archive : 연 1회 미만, 48시간 내 액세스
  - 검색 옵션
    - Standard : 3-5시간 내 완료
    - Bulk : 5-12시간 내 완료
    - Expedited : 1-5분 내 완료
  - 3개월 이내 삭제 시 조기 삭제 비용이 청구됨
- S3 Inentory : 객체 메타데이터 확인, 보고
- OAI(Origin Access Identity) : CloudFront에서 제공하는 S3로의 접근 방법
  - S3의 Principal 정책을 통해 참조됨
- 다른 계정으로 S3 복제 : 복제할 계정 관점에서 복제해야 함

## SQS, SNS

- SQS(Simple Queue Service) : 구독해서 큐 생성 가능
- SQS 대기열
  - 표준 : 순서가 바뀔 수 있지만, 초당 API 호출 수에 제한이 거의 없음
  - FIFO : 순서가 보장되지만, 초당 API 호출 수가 제한됨
- SQS 메시지 수명 주기
  - 1~14일 동안 유지(기본값 : 4일)
  - 가시성 시간 제한 : 메시지가 수신되면 다른 소비자가 다시 처리하지 못하도록 하는 시간(기본값 : 30초, 최대 12시간)
    - SQS 메시지를 수신한 소비자가 메시지를 대기열에서 삭제해야 함
    - 가시성 시간 제한이 만료되기 전에 소비자가 메시지를 삭제하지 않으면 다른 소비자가 메시지를 볼 수 있게 됨
- SNS(Simple Notification Service) : 주제 생성해서 적용 가능
- SNS : 푸시 메커니즘을 사용해서 알림 전달
  - 전송 후 회수 불가

## DB 서비스

- RDS : 관리형 서비스, RR, 스케일링 등을 직접 구축해야 함
  - DB engine : MySQL, MariaDB, Oracle, SQL Server, PostgreSQL
  - RDS Proxy : 커넥션 관리(람다는 프로비저닝때문에 커넥션을 많이 잡아먹음)
    - Secret Manager 설정을 해줘야 함
  - Aurora : 완전 관리형 RDBMS
    - DB engine : MySQL, PostgreSQL
    - Aurora Serverless : Aurora를 서버리스로 구성해서 일반 인스턴스보다 빠르지만 비용 높음
- DynamoDB : 완전 관리형 서비스
  - NoSQL → key-value 형태
  - TTL 설정 가능
  - DAX(DynamoDB Accelerator) : 인메모리 캐시 서비스
- 세션 저장 방법
  - sticky session : 트래픽 치우칠 수 있음
  - DocumentDB, DynamoDB : 영구 저장
    - DynamoDB는 Spring Session과 호환 안됨
  - ElastiCache

## 대용량 데이터 처리

- Kinesis Data Stream : 데이터를 수집 및 캡처
  - 대용량 스트리밍
- Kinesis Data Firehose : 데이터를 분석 및 모니터링
- Kinesis Data Analytics : SQL 쿼리, 필터링, 변환, 요약
- Athena : streaming data의 1회성 쿼리할 때 최적
  - Kinesis : 연속 쿼리
- Lake Formation : data lake 권한 관리 서비스
  - 데이터 저장(S3), 쿼리(Athena), metadata 저장(Glue Data Catalog) 필요
  - Data lake : 빅데이터를 위한 모든 데이터를 담는 data storage
  - Data warehouse : 배치, BI(Business Intelligence)를 위한 structured 데이터를 담는 data storage
- Redshift : data warehouse 서비스
  - S3로 지속적 자동 백업
  - 열 기반 스토리지 사용
    - 데이터 집계, 분석적 쿼리에 적함(트랜잭션에는 부적합)
  - 동시성 스케일링 : read query가 증가할 경우 자동으로 scale in/out
- Apache Parquet : 컬럼 기반 파일 저장 포멧
  - 보통 방대한 양의 데이터 저장
  - 컬럼 기반이기 때문에 유사한 데이터끼리 묶어서 저장
    - 압축률이 좋아짐
    - 컬럼별로 적합한 인코딩 사용 가능

## NAT

- NAT, Internet G/W : 인터넷 사용
- NAT 인스턴스 : NAT를 수행하도록 구성된 일반 AMI
  - 인스턴스 유형, 크기에 따라 비용 청구
  - 보안그룹 연결 가능
- NAT 게이트웨이 : 고가용성
  - 게이트웨이를 통해 보내는 데이터 양에 따라 비용 청구
  - 보안그룹 연결 불가(NAT 게이트웨이 기반 리소스와 연결해야 함)

## 연결, 보안

- ALB : URL을 통해 HTTP 상태 확인 가능
- HTTPS 인증서 설정
  1. 엔드포인트 생성
  2. ACM(AWS Certificate manager)으로 인증서 가져옴
  3. 엔드포인트에 인증서 연결
  4. 엔드포인트로 트래픽 라우팅
- 네트워크 ACL : VPC에 있는 subnet과 연결해야 함
  - 서브넷이 ACL을 명시하지 않은 경우 기본 네트워크 ACL에 연결됨
  - 사용자가 생성한 ACL은 기본적으로 모든 인바운드, 아웃바운드 트래픽 거부
  - 낮은 번호의 규칙부터 먼저 적용됨 → 좁은 범위의 설정을 먼저 해야함
- WAF : SQL Injection, XSS 등의 공격 방어
  - web ACL로 생성해서 AWS resources에 적용 가능
    - API Gateway와 함께 사용 가능
- Shield Advanced : 3 - 7 계층의 DDoS 방어
  - NLB와 함께 사용 가능

## 비용, 보고 관련

- Cost Explorer : AWS 비용, 사용 내역에 대한 자료 제공
- QuickSight : data visualization service
- Workload Discovery on AWS : AWS 워크로드 시각화
  - 워크로드 아키텍처 다이어그램 생성 가능
- Config : AWS 자원 구성 변경 추적
- CloudTrail : AWS 자원에 대한 API 호출 추적

## 파일, 온프레미스 관련

- FSx for Windows File Server : Windows 파일 시스템 + HA 스토리지
- FSx for Lustre : Linux 환경의 HPC, AI/ML에서 사용할 수 있는 파일 시스템
  - NFS 지원 x
- DataSync : 온프레미스, AWS 스토리지 서비스 사이 데이터 이동 자동, 가속화
- Direct Connect : 온프레미스 환경과 AWS 클라우드 자원을 전용 회선으로 연결
  - DX Location이 필요함
  - 호스팅 연결 : DX Location의 APN(Access Point Name 파트너의 가상 연결로 통신)
- Storage Gateway(File Gateway) : 온프레미스 스토리지를 매핑해서 백업하면서 로컬 액세스 가능

## 외부 SaaS 통합 관련

- EventBridge : 설정한 AWS 서비스 또는 SaaS 앱의 이벤트를 감지해서 다른 이벤트 호출
- Step Functions : AWS 자원들의 수행 순서를 설정 가능(lambda 함수들의 수행 순서 등)
- AppFlow : SaaS, AWS 서비스 간의 데이터 전송 통합 서비스

## 특수

- Textract : 텍스트 추출
- Comprehend Medical : 텍스트에서 PHI(Protected Health Information) 식별
- Transcribe : 음성에 대해 화자 인식 가능
- Pinpoint : 모바일 앱 푸시, SMS 등의 서비스
- Macie : ML 기반 민감한 데이터 검색, 분류, 보호
