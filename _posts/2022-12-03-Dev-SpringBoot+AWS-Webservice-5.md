---
layout: single
title: "Spring Boot와 AWS로 혼자 구현하는 웹 서비스 5 - AWS EC2, RDS(Ch.06 - 08)"
categories:
  - Dev
tags:
  - Spring Boot 2
  - PUTTY
  - AWS
  - EC2
  - Security group
  - RDS
  - MySQL
  - Deployment
---

- 트래픽이 특정 시간에만 몰리는 등의 상황에는 유동적으로 사용을 조절할 수 있는 클라우드 서비스가 유리함
- 클라우드의 형태
  - IaaS(Infrastructure as a Service) : 물리 장비를 미들웨어와 함께 묶어둔 추상화 서비스
    - VM, 스토리지, 네트워크, OS 등의 IT 인프라를 대여해줌
    - AWS EC2, S3, …
  - PaaS(Platform as a Service) : IaaS를 한 단계 더 추상화한 서비스
    - 많은 기능이 자동화됨
    - AWS Beanstalk, Heroku, …
  - SaaS(Software as a Service) : 소프트웨어 서비스
    - Google drive, Dropbox, …
- 예제에서는 AWS의 IaaS를 사용함
  - 레퍼런스 많음
  - PaaS인 Beanstalk은 프리티어로 무중단 배포가 불가능함

### EC2 인스턴스 생성

- EC2(Elastic Computer Cloud) : 성능, 용량 등을 유동적으로 사용할 수 있는, AWS에서 제공하는 서버
  - 프리티어는 t2.micro, 월 750시간 제한 존재
- 회원가입 후 사용자 그룹 생성, IAM 사용자로 로그인 후 서울 리전으로 변경
- EC2 instance 생성
  - Name : sa-web-public
  - AMI : amazon linux
  - Instance : t2.micro
  - Key pair : ec2-public
  - Security group
    - Inbound : (ssh, my IP), (custom tcp, 0000), (https, 0000)
  - Configure storage : 30GiB
- eip 할당 후 sa-web-public과 연결
  - Name : sa-web-public-eip

### EC2 서버 접속

- PUTTYgen에 pem 넣어서 ppk 받음
- PUTTY 연결
  - host name : `ec2-user@eip`
  - port : 22
  - connection-ssh-auth-private key : ec2-public.ppk

### 서버 설정

- `sudo yum install java-11-amazon-corretto.x86_64`로 자바 11 설치
- 아래 명령어로 timezone 변경
  ```bash
  sudo rm /etc/localtime
  sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
  ```
- `sudo vim /etc/sysconfig/network`에서 `HOSTNAME=sa-web` 추가

## AWS RDS

- parameter group 생성
  - Parameter group family : mariadb10.6
  - Group name : sa-db-group
  - Edit parameters
    - timezone : Asia/Seoul
    - character_set\_\* : utf8mb4
    - collation\_\* : utf8mb4_general_ci
    - max_connections : 150
- security group 생성
  - Inbound rule : (MYSQL/Aurora, My ip), (MYSQL/Aurora, sa-public-sg)
- DB 생성
  - Engine option : MariaDB
  - DB instance identifier : sa-web-dbInstance
  - Master username : admin
  - pw : qwer1234
  - Compute resource : Don’t conect ~
  - Initial database name : sa_web_db
  - DB parameter group : sa-db-group
  - publicly access : yes
- RDS endpoint : `sa-web-dbinstance.cqafqb7woojr.ap-northeast-2.rds.amazonaws.com`

### Database 플러그인 설치

- Plugins에서 Database Navigator 설치
- Database browser 실행, MySQL 추가
  - Host에 RDS endpoint 넣으면 됨
  - Apply-Ok
- Open SQL Console - New console
  - 쿼리는 드래그해서 Execute Statement 클릭

```sql
use sa_web_db;
show variables like 'c%';

ALTER DATABASE sa_web_db
CHARACTER SET = 'utf8mb4'
COLLATE = 'utf8mb4_general_ci';

SET character_set_filesystem = 'utf8mb4';
SET character_set_results = 'utf8mb4';
SET character_set_server = 'utf8mb4';
SET character_set_system = 'utf8mb4';
SET collation_server = 'utf8mb4_general_ci';

create TABLE test (
id bigint(20) NOT NULL AUTO_INCREMENT,
content varchar(255) DEFAULT null,
primary key (id)
) ENGINE=InnoDB;

insert into test(content) values ('테스트');
select * from test;
```

### EC2에서 RDS 접근 확인

```bash
# mysql 설치
sudo yum install mysql

# RDS 접속
mysql -u admin -p -h {RDS endpoint}

# DB 확인
show databases;
```

## EC2에 프로젝트 배포

- `java --version`으로 자바 버전 확인

  - 없을 경우 설치

    ```bash
    # 설치 가능한 jdk 확인
    yum list java*jdk-devel

    # aws coreetto 다운로드
    sudo curl -L https://corretto.aws/downloads/latest/amazon-corretto-11-x64-linux-jdk.rpm -o jdk11.rpm

    # jdk11 설치
    sudo yum localinstall jdk11.rpm

    # jdk version 선택
    sudo /usr/sbin/alternatives --config java

    # java 버전 확인
    java --version

    # 다운받은 설치키트 제거
    rm -rf jdk11.rpm

    # yum 설치 리스트 확인
    yum list installed | grep "java"
    # java-1.8.0-openjdk-headless.x86_64
    # java-11-amazon-corretto-devel.x86_64

    sudo yum remove java-1.8.0-openjdk-headless.x86_64
    ```

- EC2에 git 설치, 레포 클론 후 테스트 수행

  ```bash
  sudo yum install git

  mkdir ~/app && mkdir ~/app/step1
  cd ~/app/step1

  git clone https://github.com/siriyaoff/springaws.git
  cd springaws
  chmod -vR 755 *

  ./gradlew test
  ```

  - `BUILD SUCCESSFUL` 확인

### 배포 스크립트 생성

- `app/step1/deploy.sh` 생성

  ```bash
  #!/bin/bash

  REPOSITORY=/home/ec2-user/app/step1
  PROJECT_NAME=springaws

  cd $REPOSITORY/$PROJECT_NAME/

  echo "> Git Pull"

  git pull

  echo "> 프로젝트 Build 시작"

  ./gradlew build

  echo "> step1 디렉토리로 이동"

  cd $REPOSITORY

  echo "> Build 파일 복사"

  cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

  echo "> 현재 구동중인 애플리케이션 pid 확인"

  CURRENT_PID=$(pgrep -f ${PROJECT_NAME}.*.jar)

  echo "현재 구동중인 앱 pid: $CURRENT_PID"

  if [ -z "$CURRENT_PID" ]; then
          echo "> 현재 구동 중인 앱이 없으므로 종료하지 않습니다."
  else
          echo "> kill -15 $CURRENT_PID"
          kill -15 $CURRENT_PID
          sleep 5
  fi

  echo "> 새 앱 배포"

  JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)

  echo "> JAR Name: $JAR_NAME"

  nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
  ```

  - `tail -n 1` : 여러 jar 중 가장 마지막 jar 파일 선택

- `deploy.sh`에 권한 추가
  ```bash
  chmod +x ./deploy.sh
  ```
- 현재 실행하면 ClientRegistrationRepository could not be found라고 에러가 발생
  - `application-oauth.properties`가 없기 때문
  - 서버에 직접 생성해야 함

### 외부 security 파일 등록

- `/home/ec2-user/app` 디렉토리에 `application-oauth.properties` 파일 생성

  ```
  # Naver

  # registration
  spring.security.oauth2.client.registration.naver.client-id= // CLIENTID
  spring.security.oauth2.client.registration.naver.client-secret= // CLIENTSECRET
  spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
  spring.security.oauth2.client.registration.naver.authorization_grant_type=authorization_code
  spring.security.oauth2.client.registration.naver.scope=name,email,profile_image

  # provider
  spring.security.oauth2.client.provider.naver.authorization_uri=https://nid.naver.com/oauth2.0/authorize
  spring.security.oauth2.client.provider.naver.token_uri=https://nid.naver.com/oauth2.0/token
  spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
  spring.security.oauth2.client.provider.naver.user_name_attribute=response

  # Google

  spring.security.oauth2.client.registration.google.client-id= // CLIENTID
  spring.security.oauth2.client.registration.google.client-secret= // CLIENTSECRET
  spring.security.oauth2.client.registration.google.scope=profile,email
  ```

- `deploy.sh`에 프로퍼티 파일 적용 코드 추가
  ```bash
  ...
  nohup java -jar \
          -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
          $REPOSITORY/$JAR_NAME 2>&1 &
  ```
  - `-Dspring.config.location` : 스프링 설정 파일 위치 지정
    - `classpath`가 붙으면 jar 안의 resources 디렉토리를 기준으로 경로가 생성됨
    - `application-oauth.properties`는 절대경로 사용(파일이 외부에 있기 때문)

### 스프링 부트 프로젝트로 RDS 접근

- 아래와 같은 작업들이 필요함
  - 테이블 생성 : MariaDB에서는 쿼리를 이용해 직접 테이블을 생성해야 함
  - 프로젝트 설정 : Java 프로젝트가 MariaDB에 접근하기 위해선 DB 드라이버를 프로젝트에 추가해야 함
  - EC2 설정 : DB 접속 정보를 EC2 서버 내부에서 관리하도록 설정
- JPA가 사용될 엔티티 테이블, Spring 세션이 사용될 테이블 총 2가지를 생성해야 함

  - JPA가 사용할 테이블은 테스트 코드 수행 시 로그로 생성되는 쿼리를 사용하면 됨
    ```sql
    use sa_web_db;
    create table posts (id bigint not null auto_increment, created_date datetime(6), modified_date datetime(6), author varchar(255), content TEXT not null, title varchar(500) not null, primary key (id)) engine=InnoDB;
    create table users (id bigint not null auto_increment, created_date datetime(6), modified_date datetime(6), email varchar(255) not null, name varchar(255) not null, picture varchar(255), role varchar(255) not null, primary key (id)) engine=InnoDB;
    ```
  - 프로젝트에서 `schema-mysql.sql` 찾아서 거기서 테이블 생성하는 쿼리도 RDS에 반영

    ```sql
    CREATE TABLE SPRING_SESSION (
    	PRIMARY_ID CHAR(36) NOT NULL,
    	SESSION_ID CHAR(36) NOT NULL,
    	CREATION_TIME BIGINT NOT NULL,
    	LAST_ACCESS_TIME BIGINT NOT NULL,
    	MAX_INACTIVE_INTERVAL INT NOT NULL,
    	EXPIRY_TIME BIGINT NOT NULL,
    	PRINCIPAL_NAME VARCHAR(100),
    	CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
    ) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

    CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
    CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
    CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

    CREATE TABLE SPRING_SESSION_ATTRIBUTES (
    	SESSION_PRIMARY_ID CHAR(36) NOT NULL,
    	ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
    	ATTRIBUTE_BYTES BLOB NOT NULL,
    	CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
    	CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
    ) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
    ```

- `build.gradle`에 `implementation 'org.mariadb.jdbc:mariadb-java-client'` 등록
- `resources`에 `application-real.properties` 생성
  ```
  spring.profiles.include=oauth,real-db
  spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
  spring.session.store-type=jdbc
  server.servlet.encoding.charset=UTF-8
  server.servlet.encoding.force=true
  ```
  - 이후 깃헙에 푸시
- EC2의 `app` 디렉토리에 `application-real-db.properties` 파일 생성
  ```
  spring.jpa.hibernate.ddl-auto=none
  spring.datasource.hikari.jdbc-url=jdbc:mariadb://sa-web-dbinstance.cqafqb7woojr.ap-northeast-2.rds.amazonaws.com:3306/sa_web_db
  spring.datasource.username=admin
  spring.datasource.password=qwer1234
  spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
  ```
  - `spring.jpa.hibernate.ddl-auto=none`를 설정해주지 않으면 테이블이 모두 새로 생성될 수 있음
- `deploy.sh`에서 real profile을 사용할 수 있도록 적용
  ```bash
  nohup java -jar \
          -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties \
          -Dspring.profiles.active=real \
          $REPOSITORY/$JAR_NAME 2>&1 &
  ```
  - `-Dspring.profiles.active=real \` : `application-real.properties`를 활성화시킴
    - real profile 내부의 `spring.profiles.include=oauth,real-db` 때문에 `real-db`도 활성화됨
- 배포 실행 후 로그에서 톰캣 실행된 것 확인
- `curl localhost:8080`으로 페이지 뜨는거 확인

### EC2에서 소셜 로그인하기

- EC2 보안 그룹에 (CustomTCP, 8080, anywhere) 추가
- public DNS를 통해 접속 가능
  - [http://ec2-15-164-249-30.ap-northeast-2.compute.amazonaws.com:8080/](https://ec2-15-164-249-30.ap-northeast-2.compute.amazonaws.com:8080/)
- 구글에 EC2 주소 등록
  - cloud console - 프로젝트 대시보드 - 사용자 인증 정보 - OAuth 2.0 클라이언트 ID - 승인된 리디렉션 URI에 EC2 로그인 성공 URI 추가
    ```
    http://ec2-15-164-249-30.ap-northeast-2.compute.amazonaws.com:8080/login/oauth2/code/google
    ```
- naver developer - my application - API 설정
  - 서비스 URL, Callback URL 수정
