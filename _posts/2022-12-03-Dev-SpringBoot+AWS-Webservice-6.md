---
layout: single
title: "Spring Boot와 AWS로 혼자 구현하는 웹 서비스 6 - CI/CD(Ch.09 - 10)"
categories:
  - Dev
tags:
  - Spring Boot 2
  - Travis CI
  - CI
  - AWS
  - Policy
  - User
  - Group
  - Role
  - S3
  - CodeDeploy
  - Nginx
---

- CI(Continuous Integration, 지속적 통합) : VCS 시스템에 PUSH되면 자동으로 테스트, 빌드가 수행되어 안정적인 배포 파일을 만드는 과정
  - VCS(Version Control System) : Git, SVN 등
- CD(Continuous Deployment, 지속적 배포) : 빌드 결과를 무중단으로 운영서버에 올리는 과정
- 일반적으로 CI만 있는 경우는 잘 없음
- CI의 조건
  - 모든 소스 코드가 실행될 수 있고, 누구든 현재의 소스에 접근할 수 있는 단일 지점을 유지할 것
  - 빌드 프로세스를 자동화해서 누구든 빌드하는 단일 명령어를 사용할 수 있게 할 것
  - 테스팅을 자동화해서 단일 명령어로 언제든지 시스템에 대한 테스트를 실행할 수 있게 할 것
  - 누구나 현재 실행파일을 얻으면 지금까지 가장 완전한 실행 파일을 얻었다는 확신을 하게 할 것
    - 프로젝트가 완전한 상태임을 보장하기 위해 테스트 코드가 구현되어 있어야 함

## Travis CI

- Github에서 제공하는 CI 서비스
  - `.travis.yml`이라는 YML파일로 설정 가능
  - YML : JSON에서 괄호를 제거한 파일이라고 생각하면 편함
- 루트 디렉토리에 `.travis.yml` 파일 추가

```yaml
language: java
jdk:
  - openjdk8

branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - "$HOME/.m2/repository"
    - "$HOME/.gradle"

script: "./gradlew clean build"

# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - "hwy16016@gmail.com"
```

- `branches` : 어느 브랜치가 푸시될 때 수행할 지 지정
- `cache` : 의존성을 디렉토리에 캐시함
- `script` : 브랜치가 푸시되었을 때 수행할 명령어
- `notifications` : CI 실행 완료 시 알림 설정

### Travis CI 연동시 아키텍처

![springawsnote3](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/springawsnote3.png)

- AWS Code Deploy는 저장 기능이 없기 때문에 Travis CI에서 빌드한 결과를 저장할 공간이 필요함
  - 보통 S3를 사용함
- 빌드 없이 배포만 필요한 경우가 있을 수 있기 때문에 빌드와 배포를 분리해서 확장성을 넓히는게 좋음

## Travis CI - AWS S3 연동

1. AWS Key 발급  
   IAM - Users - Add users

   - User name : springaws-travis-deploy
   - AWS credential type : Access key - Programmatic access
   - Set permissions : Attach existing policies directly
     - AmazonS3FullAccess
     - AWSCodeDeployFullAccess
   - 이 강의에서는 간단하게 S3, CodeDeploy를 같이 관리함

2. Travis CI에 키 등록  
   repo Settings - Environment Variables
   - AWS_ACCESS_KEY : aws key
     - Branch : master
   - AWS_SECRET_KEY : aws secret key
     - Branch : master
   - `.travis.yml`에서 `$AWS_ACCESS_KEY`와 같이 변수로 사용할 수 있음
3. S3 버킷 생성  
   S3 - Create bucket

   - Bucket name : smspringaws-springboot-build
     - globally unique 해야 함
   - ACLs disabled
   - Block all public access : block

4. `.travis.yml` 추가

```yaml
---
before_deploy:
  - zip -r springaws-springboot2-webservice *
  - mkdir -p deploy
  - mv springaws-springboot2-webservice.zip deploy/springaws-springboot2-webservice.zip

deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY
    bucket: smspringaws-springboot-build
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private으로 함
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait-until-deployed: true
# CI 실행 완료 시 메일로 알람
```

- `before_deploy` : deploy 명령어가 실행되기 전에 수행됨
  - CodeDeploy는 Jar 파일을 인식하지 못하기 때문에 Jar 파일과 다른 설정파일들을 모아 압축함
  - `zip` 명령의 두 번째 파라미터는 본인 프로젝트 이름(`springaws`)이어야 함
  - `mkdir -p deploy` : deploy 디렉토리를 Travis CI가 실행중인 위치에 생성함
- `deploy` : S3로 파일 업로드, CodeDeploy로 배포 등의 외부 서비스와 연동될 작업을 선언함
  - `local_dir: deploy` : 해당 위치(`deploy`)의 파일들만 S3로 전송

## Travis CI - AWS S3, CodeDeploy 연동

### **Policy, User, Group, Role 개념**

- Policy : 권한을 부여하는 방법
  - AWS 리소스에 대해 어떤 작업을 수행할 수 있는지 JSON 형식으로 작성됨
  - IAM User, Group, Role에 연결될 수 있음
- User : AWS를 사용하는 사용자 한 명
- Group : IAM User가 속할 수 있는 집합
  - 한 사용자가 그룹에 속하면 그 그룹에 연결된 정책이 자동으로 적용됨
    - 우선 순위는 scope가 좁은 정책이 더 높음(role>user>group)
- Role : 신뢰할 수 있는 IAM User나 application 또는 AWS 서비스가 가질 수 있는 권한
  - AWS Key로 인증된 상태일 때 신뢰할 수 있다고 말함
  - 임시 보안 자격 증명 : Role을 맡게 되는 과정
  - IAM User나 Group에 영원히 연결되지 않음
- Role : AWS 서비스에만 할당할 수 있는 권한  
  e.g. EC2, CodeDeploy, SQS 등

### **AWS Developer Tools**

- Code Commit : 코드 저장소(Github과 비슷)
- Code Build : 빌드용 서비스(Travis CI)
  - 멀티 모듈일 경우에도 보통 젠킨스/팀시티 등을 사용함
- CodeDeploy : 배포 서비스
  - 대체재가 없음
  - Auto scaling, Blue green, Rolling, EC2 단독 배포 등 많은 기능 지원

1.  EC2에 IAM role 추가  
    IAM - Roles - Create role

    - Trusted entity type : AWS service
    - Use case : EC2
    - Permissions policies : AmazonEC2RoleforAWSCodeDeploy
    - Role name : ec2-codedeploy-role
    - tags : (Name, ec2-codedeploy-role)
    - Instance 선택 후 우클릭 - Security - Modify IAM role
      - IAM role : ec2-codedeploy-role
      - EC2 재부팅해야 적용됨

2.  CodeDeploy 에이전트 설치  
     EC2(sm-sa-web) 접속 후 아래 명령어 실행

    ```bash
    aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2
    chmod +x ./install
    sudo ./install auto
    sudo service codedeploy-agent status
    ```

    - `ruby: No such file or directory` 에러 뜨면 `sudo yum install ruby`로 루비 설치
    - 마지막 명령 실행 후 AWS CodeDeploy agent의 PID가 정상적으로 나와야 함

3.  CodeDeploy를 위한 권한 생성  
    IAM - Roles - Create role
    - Use case : CodeDeploy
    - Role name : codedeploy-role
    - tags : (Name, codedeploy-role)
4.  CodeDeploy 생성  
    Developer Tools - CodeDeploy - Applications - Create application

    - Application name : springaws-springboot2-webservice
    - Compute platform : EC2/On-premises

    Create deployment group

    - Deployment group name : springaws-springboot2-webservice-group
    - Service role : codedeploy-role
    - Deployment type : In-place
      - 배포할 서비스가 2대 이상일 경우 Blue/gree 선택
    - Environment configuration : Amazon EC2 instances
    - Tag group1 : (Name, sa-web-public)
      - tag group들에 의해 식별된 인스턴스가 배포됨
    - Deployment settings : CodeDeployDefault.AllAtOnce
      - 2대 이상일 경우 나눠서 배포하는 옵션을 선택할 수 있음
      - 지금은 1대이기 때문에 한 번에 배포하는 옵션 선택
    - Load balancer : Uncheck(Disable load balancing)

5.  Travis CI, S3, CodeDeploy 연동

    - EC2 instance sa-web-public 들어가서 `/home/ec2-user/app/step2/zip/` 디렉토리 생성
    - CodeDeploy 설정을 위해서 springaws 루트 디렉토리에 `appspec.yml` 생성
      ```yaml
      version: 0.0
      os: linux
      files:
        - source: /
          destination: /home/ec2-user/app/step2/zip/
          overwrite: yes
      ```
      - `version` : CodeDeploy 버전
      - `source` : CodeDeploy에 전달해준 파일 중 destination으로 이동시킬 대상을 지정
        - `/`를 입력하면 전체 파일 지정
      - `destination` : 지정된 파일을 받을 위치
    - `.travis.yml`에도 CodeDeploy 관련 설정 추가

      ```yaml
      ...
      		wait-until-deployed: true

        - provider: codedeploy
          access_key_id: $AWS_ACCESS_KEY
          secret_access_key: $AWS_SECRET_KEY
          bucket: smspringaws-springboot-build
          key: springaws-springboo2-webservice.zip # 빌드 파일을 압축해서 전달
          bundle_type: zip
          application: springaws-springboot2-webservice # CodeDeploy application 이름
          deployment_group: springaws-springboot2-webservice-group # Deployment group 이름
          region: ap-northeast-2
          wait-until-deployed: true

      # CI 실행 완료 시 메일로 알람
      ...
      ```

## 배포 자동화 구성

1. `scripts/deploy.sh` 파일 추가

   ```bash
   #!/bin/bash

   REPOSITORY=/home/ec2-user/app/step2
   PROJECT_NAME=springaws

   echo "> Build 파일 복사"

   cp $REPOSITORY/zip/*.jar $REPOSITORY/

   echo "> 현재 구동중인 애플리케이션 pid 확인"

   CURRENT_PID=$(pgrep -fl springaws | grep jar | awk '{print $1}')

   echo "현재 구동중인 앱 pid: $CURRENT_PID"

   if [ -z "$CURRENT_PID" ]; then
           echo "> 현재 구동 중인 앱이 없으므로 종료하지 않습니다."
   else
           echo "> kill -15 $CURRENT_PID"
           kill -15 $CURRENT_PID
           sleep 5
   fi

   echo "> 새 애플리케이션 배포"

   JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

   echo "> JAR Name: $JAR_NAME"

   echo "> $JAR_NAME 에 실행권한 추가"

   chmod +x $JAR_NAME

   echo "> $JAR_NAME 실행"

   nohup java -jar \
           -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties \
           -Dspring.profiles.active=real \
           $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
   ```

   - `git pull`으로 직접 빌드했던 부분 삭제
   - `CURRENT_PID=~` : springaws 이름을 가진 jar를 찾은 뒤 ID를 찾음
   - Jar 파일은 실행 권한이 없는 상태이기 때문에 nohup으로 실행할 수 있게 직접 권한을 추가해줘야 함
   - `$JAR_NAME > $REPOSITORY/nohup.out 2>&1 &`
   - nohup이 끝나기 전에는 CodeDeploy도 끝나지 않고 무한 대기함
   - 이를 해결하기 위해 nohup.out을 표준 입출력으로 사용해 CodeDeploy 로그에 출력함

2. 배포에 필요한 파일들만 전송하도록 `.travis.yml` 파일 수정

   ```yaml
   ---
   before_deploy:
     - mkdir -p before-deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
     - cp scripts/*.sh before-deploy/
     - cp appspec.yml before-deploy/
     - cp build/libs/*.jar before-deploy/
     - cd before-deploy && zip -r before-deploy * # before-deploy로 이동 후 전체 압축
     - cd ../ && mkdir -p deploy # 상위 디렉토리로 이동 후 deploy 디렉토리 생성
     - mv before-deploy/before-deploy.zip deploy/springaws-springboot2-webservice.zip # deploy로 zip 파일 이동
   ```

   - Travis CI에서는 S3로 특정 파일만 업로드를 지원하지 않음
     - 디렉토리 단위로만 업로드가 가능함  
       → before-deploy 디렉토리는 항상 생성해야 함

3. `appspec.yml`에 권한, hook 설정 추가

   ```yaml
   ---
   permissions:
     - object: /
       pattern: "**"
       owner: ec2-user
       group: ec2-user

   hooks:
     ApplicationStart:
       - location: deploy.sh
         timeout: 60
         runas: ec2-user
   ```

   - `permissions` : CodeDeploy에서 EC2로 넘겨준 파일들 모두에 ec2-user의 권한 추가
   - `hooks` : CodeDeploy 배포 단계에서 실행할 명령어 지정
     - `ApplicationStart` 단계에서 `deploy.sh`를 `ec2-user` 권한으로 실행함

- CodeDeploy에서 배포 결과 확인 가능
  ![springawsnote4](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/springawsnote4.png)

## CodeDeploy 로그 확인

- `/opt/codedeploy-agent/deployment-root`에서 로그 확인 가능
  ```yaml
  [ec2-user@ip-172-31-41-214 ccc17b75-6021-4d6e-b31e-58da053cf0f9]$ cd /opt/codedeploy-agent/deployment-root/
  [ec2-user@ip-172-31-41-214 deployment-root]$ ll
  total 0
  drwxr-xr-x 5 root root  63 Nov 26 02:15 ccc17b75-6021-4d6e-b31e-58da053cf0f9
  drwxr-xr-x 2 root root 247 Nov 26 02:15 deployment-instructions
  drwxr-xr-x 2 root root  46 Nov 26 02:15 deployment-logs
  drwxr-xr-x 2 root root   6 Nov 26 02:16 ongoing-deployment
  [ec2-user@ip-172-31-41-214 deployment-root]$ cd ccc17b75-6021-4d6e-b31e-58da053cf0f9/
  [ec2-user@ip-172-31-41-214 ccc17b75-6021-4d6e-b31e-58da053cf0f9]$ ll
  total 0
  drwxr-xr-x 2 root root 24 Nov 26 01:30 d-4PMS7NKDK
  drwxr-xr-x 4 root root 62 Nov 26 02:15 d-9XO2VQKDK
  drwxr-xr-x 3 root root 50 Nov 26 01:34 d-RDLFOIKDK
  [ec2-user@ip-172-31-41-214 ccc17b75-6021-4d6e-b31e-58da053cf0f9]$ cd d-4PMS7NKDK/
  [ec2-user@ip-172-31-41-214 d-4PMS7NKDK]$ ll
  total 0
  -rw-r--r-- 1 root root 0 Nov 26 01:30 bundle.tar
  [ec2-user@ip-172-31-41-214 d-4PMS7NKDK]$ cd ../..
  [ec2-user@ip-172-31-41-214 deployment-root]$ cd deployment-logs/
  [ec2-user@ip-172-31-41-214 deployment-logs]$ ll
  total 4
  -rw-r--r-- 1 root root 987 Nov 26 02:16 codedeploy-agent-deployments.log
  ```
  - CodeDeploy ID가 이름을 설정된 디렉토리 내부에는 배포한 단위 별로 배포 파일들이 저장됨
  - `deployment-logs/codedeploy-agent-deployments.log`에 로그가 저장됨(위에서 설정했던 nohup 로그 포함)

## 무중단 배포

- 이전까지는 새로운 Jar가 실행되기 전까지 기존의 Jar를 종료시켜 놓기 때문에 서비스가 중단됨
- 무중단 배포의 종류
  - AWS Blue-Green 배포
  - 도커
  - Nginx
  - L4 스위치

### Nginx

- 서버 하나에 Nginx 1대와 Spring Boot Jar 2대를 사용하는 방식

![springawsnote5](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/springawsnote5.png)

- Nginx : 80(http), 443(https) 포트 사용
- Jar : 각각 8081, 8082 포트 사용
- 무중단 배포 과정
  1. nginx는 8081을 바라보게한 뒤 8082의 서비스를 배포 후 구동 테스트
  2. 정상 작동하면 nginx reload로 8082를 바라보게 한 뒤 8081도 배포
  3. 정상 작동 확인 후 다시 8081로 변경

### Nginx 설치와 Spring Boot 연동

1. 아래 명령어로 EC2에 nginx 설치 후 확인

   ```yaml
   sudo amazon-linux-extras install nginx1
   sudo service nginx start
   ```

2. EC2 Security Group(sa-web-sg)에 80포트 추가
   - Inbound rule, (TCP 80, Anywhere-IPv4)
3. OAuth에 redirection 주소 추가
   - 구글 : 승인된 리디렉션 URI에 `http://ec2-3-35-198-115.ap-northeast-2.compute.amazonaws.com/login/oauth2/code/google` 추가
   - 네이버 : Callback URL에 `http://ec2-3-35-198-115.ap-northeast-2.compute.amazonaws.com/login/oauth2/code/naver` 추가
   - EC2의 도메인으로 포트 번호 없이 접속해서 nginx 웹페이지가 출력되는지 확인
4. nginx와 spring boot 연동

   - `sudo vim /etc/nginx/nginx.conf`로 nginx 설정 파일 열어서 아래 내용 업데이트

     ```
     ...
     include /etc/nginx/default.d/*.conf;

     location / {
     				proxy_pass http://localhost:8080;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header Host $http_host;
     }

     error_page 404 /404.html;
     ...
     ```

     - `proxy_pass http://localhost:8080;` : nginx로 요청이 오면 `localhost:8080`으로 전달
     - `proxy_set_header ~` : 실제 요청 데이터를 request header의 각 항목들(`X-Real-IP` 등)에 할당함

   - `sudo service nginx restart`로 nginx 재시작 후 EC2 도메인으로 포트 번호 없이 접속해서 웹페이지 뜨는지 확인

### 무중단 배포 스크립트 만들기

1. profile API 추가

   - `ProfileController` 생성

     ```java
     @RequiredArgsConstructor
     @RestController
     public class ProfileController {
         private final Environment env;

         @GetMapping("/profile")
         public String profile() {
             List<String> profiles = Arrays.asList(env.getActiveProfiles());
             List<String> realProfiles = Arrays.asList("real", "real1", "real2");
             String defaultProfile = profiles.isEmpty() ? "default" : profiles.get(0);

             return profiles.stream()
                     .filter(realProfiles::contains)
                     .findAny()
                     .orElse(defaultProfile);
         }
     }
     ```

     - `env.getActiveProfiles()` : 현재 실행중인 Active Profile을 모두 가져옴
       - real, oauth, real-db 등
       - real, real1, real2는 모두 배포에 사용될 profile이기 때문에 이 중 하나라도 있으면 그 값을 반환함
       - 무중단 배포에는 real1, real2만 사용되지만, step2의 real도 일단 포함시켜둠

   - 테스트 코드 작성

     ```java
     public class ProfileControllerUnitTest {
         @Test
         public void get_real_profile() {
             //given
             String expectedProfile = "real";
             MockEnvironment env = new MockEnvironment();
             env.addActiveProfile(expectedProfile);
             env.addActiveProfile("oauth");
             env.addActiveProfile("real-db");

             ProfileController controller = new ProfileController(env);

             //when
             String profile = controller.profile();

             //then
             assertThat(profile).isEqualTo(expectedProfile);
         }

         @Test
         public void getFirstWhenNo_real_profile() {
             //given
             String expectedProfile = "oauth";
             MockEnvironment env = new MockEnvironment();

             env.addActiveProfile(expectedProfile);
             env.addActiveProfile("real-db");

             ProfileController controller = new ProfileController(env);

             //when
             String profile = controller.profile();

             //then
             assertThat(profile).isEqualTo(expectedProfile);
         }

         @Test
         public void getDefaultWhenNo_active_profile() {
             //given
             String expectedProfile = "default";
             MockEnvironment env = new MockEnvironment();
             ProfileController controller = new ProfileController(env);

             //when
             String profile = controller.profile();

             //then
             assertThat(profile).isEqualTo(expectedProfile);
         }
     }
     ```

     - `Environment`는 인터페이스이기 때문에 `MockEnvironment`를 사용해서 테스트할 수 있음
       - 생성자 DI로 받았기 때문에 가능
       - `@Autowired`로 받았다면 다른 방법으로 테스트 코드를 구현해야 함
     - 통과하는지 확인

   - `SecurityConfig`에 제외 코드 추가
     ```java
     ...
     .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**", "/profile").permitAll()
     ...
     ```
   - `SecurityConfig` 설정 테스트 추가

     ```java
     @ExtendWith(SpringExtension.class)
     @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
     public class ProfileControllerTest {
         @LocalServerPort
         private int port;

         @Autowired
         private TestRestTemplate restTemplate;

         @Test
         public void callprofilewithoutAuth() throws Exception {
             String expected = "default";

             ResponseEntity<String> response = restTemplate.getForEntity("/profile", String.class);
             assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
             assertThat(response.getBody()).isEqualTo(expected);
         }
     }
     ```

     - 통과하는지 확인 후 깃헙으로 푸시

   - 배포가 끝난 후 `/profile`로 접속해서 profile이 나오는지 확인

2. real1, real2 profile 생성

   ```java
   // application-real1.properties
   server.port=8081
   spring.profiles.include=oauth, real-db
   spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
   spring.session.store-type=jdbc
   server.servlet.encoding.charset=UTF-8
   server.servlet.encoding.force=true

   // application-real2.properties
   server.port=8082
   spring.profiles.include=oauth, real-db
   spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
   spring.session.store-type=jdbc
   server.servlet.encoding.charset=UTF-8
   server.servlet.encoding.force=true
   ```

3. nginx 설정 수정

   - `sudo vim /etc/nginx/conf.d/service-url.inc`로 파일 생성 후 `set $service_url http://127.0.0.1:8080;` 내용 추가
   - `sudo vim /etc/nginx/nginx.conf`로 설정 파일 열어서 아래 내용 수정

     ```
     ...
     include /etc/nginx/default.d/*.conf;
     include /etc/nginx/conf.d/service-url.inc;

     location / {
     				proxy_pass $service_url;
     ...
     ```

   - `sudo service nginx restart`로 nginx 재시작

4. 배포 스크립트들 작성

   - EC2에 `step3`, `step3/zip` 디렉토리 생성
     - 무중단 배포는 `step3` 사용 예정
   - `appspec.yml` 경로 수정
     ```yaml
     files:
       - source: /
         destination: /home/ec2-user/app/step3/zip/
     ```
   - 5개의 스크립트를 통해 무중단 배포가 진행됨
     1. `stop.sh` : nginx에 연결되어 있진 않지만, 실행중이던 spring boot 종료
     2. `start.sh` : 배포할 spring boot 프로젝트를 `stop.sh`로 종료한 profile으로 실행
     3. `health.sh` : `start.sh`로 실행시킨 프로젝트가 정상 작동하는지 확인
     4. `switch.sh` : nginx가 바라보는 spring boot를 최신 버전으로 변경
     5. `profile.sh` : 앞의 스크립트들에서 공용으로 사용할 profile과 포트 확인
   - `appspec.yml`에 위 스크립트들 사용하도록 설정
     ```yaml
     hooks:
       AfterInstall:
         - location: stop.sh # nginx와 연결되지 않은 spring boot 종료
           timeout: 60
           runas: ec2-user
       ApplicationStart:
         - location: start.sh # nginx와 연결되지 않은 Port로 새 버전의 spring boot 시작
           timeout: 60
           runas: ec2-user
       ValidateService:
         - location: health.sh # 새 spring boot가 정상 작동하는지 확인
           timeout: 60
           runas: ec2-user
     ```
   - 아래 스크립트들 `scripts` 폴더에 추가

     - `profile.sh`

       ```bash
       #!/usr/bin/env bash

       # 쉬고 있는 profile 찾기: real1이 사용 중이면 real2, v.v

       function find_idle_profile()
       {
         RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)

         if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면
         then
           CURRENT_PROFILE=real2
         else
           CURRENT_PROFILE=$(curl -s http://localhost/profile)
         fi

         if [ ${CURRENT_PROFILE} == real1 ]
         then
           IDLE_PROFILE=real2
         else
           IDLE_PROFILE=real1
         fi

         echo "${IDLE_PROFILE}"
       }

       # 쉬고 있는 profile의 port 찾기
       function find_idle_port()
       {
         IDLE_PROFILE=$(find_idle_profile)

         if [ ${IDLE_PROFILE} == real1 ]
         then
           echo "8081"
         else
           echo "8082"
         fi
       }
       ```

       - `$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)` : 현재 nginx가 보고 있는 spring boot가 정상 작동 중인지 확인
         - 응답 코드가 400 이상이면 real2를 현재 profile로 사용
       - `IDLE_PROFILE` : nginx와 연결되지 않은 profile
         - `echo`로 출력하고 그 값을 다시 잡아서 사용함(`$(find_idle_profile)`과 같이)

     - `stop.sh`

       ```bash
       #!/usr/bin/env bash

       ABSPATH=$(readlink -f $0)
       ABSDIR=$(dirname $ABSPATH)
       source ${ABSDIR}/profile.sh

       IDLE_PORT=$(find_idle_port)

       echo "> $IDLE_PORT 에서 구동 중인 애플리케이션 pid 확인"
       IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

       if [ -z ${IDLE_PID} ]
       then
         echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
       else
         echo "> kill -15 $IDLE_PID"
         kill -15 ${IDLE_PID}
         sleep 15
       fi
       ```

       - `ABSDIR=$(dirname $ABSPATH)` : 현재 `stop.sh`가 속한 경로를 찾음
         - `profile.sh`의 경로를 찾기 위해 사용됨
       - `source ${ABSDIR}/profile.sh` : JAVA의 import와 비슷함
         - `profile.sh`의 함수들을 사용할 수 있게 됨

     - `start.sh`

       ```bash
       #!/usr/bin/env bash

       ABSPATH=$(readlink -f $0)
       ABSDIR=$(dirname $ABSPATH)
       source ${ABSDIR}/profile.sh

       REPOSITORY=/home/ec2-user/app/step3
       PROJECT_NAME=springaws

       echo "> Build 파일 복사"
       echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

       cp $REPOSITORY/zip/*.jar $REPOSITORY/

       echo "> 새 애플리케이션 배포"

       JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

       echo "> JAR Name: $JAR_NAME"

       echo "> $JAR_NAME 에 실행권한 추가"

       chmod +x $JAR_NAME

       echo "> $JAR_NAME 실행"

       IDLE_PROFILE=$(find_idle_profile)
       echo "> $JAR_NAME 을 profile=$IDLE_PROFILE 로 실행합니다."
       nohup java -jar \
               -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
               -Dspring.profiles.active=$IDLE_PROFILE \
               $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
       ```

       - `IDLE_PROFILE`을 통해 properties 파일을 가져오고 active profile을 지정함

     - `health.sh`

       ```bash
       #!/usr/bin/env bash

       ABSPATH=$(readlink -f $0)
       ABSDIR=$(dirname $ABSPATH)
       source ${ABSDIR}/profile.sh
       source ${ABSDIR}/switch.sh

       IDLE_PORT=$(find_idle_port)

       echo "> Health Check Start!"
       echo "> IDLE_PORT: $IDLE_PORT"
       echo "> curl -s http://localhost:$IDLE_PORT/profile"
       sleep 10

       for RETRY_COUNT in {1..10}
       do
         RESPONSE=$(curl -s http://localhost:$IDLE_PORT/profile)
         UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

         if [ ${UP_COUNT} -ge 1 ]
         then # 'real' 문자열이 있을 경우
           echo "> Health Check 성공"
           switch_proxy
           break
         else
           echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
           echo "> Health check: ${RESPONSE}"
         fi

         if [ ${RETRY_COUNT} -eq 10 ]
         then
           echo "> Health check 실패"
           echo "> nginx에 연결하지 않고 배포를 종료합니다."
           exit 1
         fi

         echo "> Health check 연결 실패.. 재시도..."
         sleep 10
       done
       ```

       - nginx 프록시 설정 변경은 `switch.sh`에서 실행함

     - `switch.sh`

       ```bash
       #!/usr/bin/env bash

       ABSPATH=$(readlink -f $0)
       ABSDIR=$(dirname $ABSPATH)
       source ${ABSDIR}/profile.sh

       function switch_proxy()
       {
         IDLE_PORT=$(find_idle_port)

         echo "> 전환할 Port: $IDLE_PORT"
         echo "> Port 전환"
         echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc

         echo "> nginx Reload"
         sudo service nginx reload
       }
       ```

       - `echo "set \$service_url http://127.0.0.1:${IDLE_PORT};"` : 하나의 문장을 pipeline으로 넘겨주기 위해 `echo`를 사용함
         - nginx가 변경할 프록시 주소를 생성함
         - double quotes(`""`)를 사용해야 함
           - 사용하지 않으면 `$service_url`을 인식하지 못함
       - `| sudo tee /etc/nginx/conf.d/service-url.inc` : 앞에서 넘겨준 문장을 `service-url.inc`에 덮어씀
       - `sudo service nginx reload` : nginx 설정을 다시 불러옴
         - `restart`와 다름
           - `restart`는 잠시 끊기지만 `reload`는 끊김없이 다시 불러옴
           - 중요한 설정들은 `reload`로 반영되지 않고 `restart`를 해야 함
           - `service-url`과 같은 외부 설정파일을 불러오는 것은 `reload`로 해결 가능

### 무중단 배포 테스트

- 자동으로 Jar의 버전 값이 변경될 수 있도록 `build.gradle` 수정

  ```groovy
  ...
  version = '1.0.1-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")
  ...
  ```

- `build.gradle`은 Groovy 기반이기 때문에 `new Date()`로 Jar 이름에 시간을 추가함
- `tail -f codedeploy-agent-deployments.log`로 CodeDeploy의 로그 확인
  ```
  [2022-11-29 00:57:34.644] [d-OC4XKAIFK]LifecycleEvent - AfterInstall
  [2022-11-29 00:57:34.644] [d-OC4XKAIFK]Script - stop.sh
  [2022-11-29 00:57:34.717] [d-OC4XKAIFK][stdout]> 8081 에서 구동 중인 애플리케이션 pid 확인
  [2022-11-29 00:57:34.761] [d-OC4XKAIFK][stdout]> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다.
  [2022-11-29 00:57:35.683] [d-OC4XKAIFK]LifecycleEvent - ApplicationStart
  [2022-11-29 00:57:35.683] [d-OC4XKAIFK]Script - start.sh
  [2022-11-29 00:57:35.699] [d-OC4XKAIFK][stdout]> Build 파일 복사
  [2022-11-29 00:57:35.699] [d-OC4XKAIFK][stdout]> cp /home/ec2-user/app/step3/zip/*.jar /home/ec2-user/app/step3/
  [2022-11-29 00:57:35.762] [d-OC4XKAIFK][stdout]> 새 애플리케이션 배포
  [2022-11-29 00:57:35.771] [d-OC4XKAIFK][stdout]> JAR Name: /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128155625.jar
  [2022-11-29 00:57:35.771] [d-OC4XKAIFK][stdout]> /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128155625.jar 에 실행권한 추가
  [2022-11-29 00:57:35.771] [d-OC4XKAIFK][stdout]> /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128155625.jar 실행
  [2022-11-29 00:57:35.809] [d-OC4XKAIFK][stdout]> /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128155625.jar 을 profile=real1 로 실행합니다.
  [2022-11-29 00:57:36.736] [d-OC4XKAIFK]LifecycleEvent - ValidateService
  [2022-11-29 00:57:36.736] [d-OC4XKAIFK]Script - health.sh
  [2022-11-29 00:57:36.936] [d-OC4XKAIFK][stdout]> Health Check Start!
  [2022-11-29 00:57:36.936] [d-OC4XKAIFK][stdout]> IDLE_PORT: 8081
  [2022-11-29 00:57:36.936] [d-OC4XKAIFK][stdout]> curl -s http://localhost:8081/profile
  [2022-11-29 00:57:46.987] [d-OC4XKAIFK][stdout]> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다.
  [2022-11-29 00:57:46.987] [d-OC4XKAIFK][stdout]> Health check:
  [2022-11-29 00:57:46.988] [d-OC4XKAIFK][stdout]> Health check 연결 실패.. 재시도...
  [2022-11-29 00:57:57.297] [d-OC4XKAIFK][stdout]> Health Check 성공
  [2022-11-29 00:57:57.351] [d-OC4XKAIFK][stdout]> 전환할 Port: 8081
  [2022-11-29 00:57:57.351] [d-OC4XKAIFK][stdout]> Port 전환
  [2022-11-29 00:57:57.379] [d-OC4XKAIFK][stdout]set $service_url http://127.0.0.1:8081;
  [2022-11-29 00:57:57.381] [d-OC4XKAIFK][stdout]> nginx Reload
  [2022-11-29 00:57:57.403] [d-OC4XKAIFK][stderr]Redirecting to /bin/systemctl reload nginx.service
  [2022-11-29 01:05:48.926] [d-2XXOO3JFK]LifecycleEvent - AfterInstall
  [2022-11-29 01:05:48.927] [d-2XXOO3JFK]Script - stop.sh
  [2022-11-29 01:05:48.978] [d-2XXOO3JFK][stdout]> 8082 에서 구동 중인 애플리케이션 pid 확인
  [2022-11-29 01:05:49.019] [d-2XXOO3JFK][stdout]> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다.
  [2022-11-29 01:05:49.976] [d-2XXOO3JFK]LifecycleEvent - ApplicationStart
  [2022-11-29 01:05:49.976] [d-2XXOO3JFK]Script - start.sh
  [2022-11-29 01:05:49.992] [d-2XXOO3JFK][stdout]> Build 파일 복사
  [2022-11-29 01:05:49.992] [d-2XXOO3JFK][stdout]> cp /home/ec2-user/app/step3/zip/*.jar /home/ec2-user/app/step3/
  [2022-11-29 01:05:50.087] [d-2XXOO3JFK][stdout]> 새 애플리케이션 배포
  [2022-11-29 01:05:50.089] [d-2XXOO3JFK][stdout]> JAR Name: /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128160441.jar
  [2022-11-29 01:05:50.089] [d-2XXOO3JFK][stdout]> /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128160441.jar 에 실행권한 추가
  [2022-11-29 01:05:50.090] [d-2XXOO3JFK][stdout]> /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128160441.jar 실행
  [2022-11-29 01:05:50.137] [d-2XXOO3JFK][stdout]> /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128160441.jar 을 profile-real2 로 실행합니다.
  [2022-11-29 01:05:51.042] [d-2XXOO3JFK]LifecycleEvent - ValidateService
  [2022-11-29 01:05:51.043] [d-2XXOO3JFK]Script - health.sh
  [2022-11-29 01:05:51.231] [d-2XXOO3JFK][stdout]> Health Check Start!
  [2022-11-29 01:05:51.231] [d-2XXOO3JFK][stdout]> IDLE_PORT: 8082
  [2022-11-29 01:05:51.231] [d-2XXOO3JFK][stdout]> curl -s http://localhost:8082/profile
  [2022-11-29 01:06:01.287] [d-2XXOO3JFK][stdout]> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다.
  [2022-11-29 01:06:01.287] [d-2XXOO3JFK][stdout]> Health check:
  [2022-11-29 01:06:01.287] [d-2XXOO3JFK][stdout]> Health check 연결 실패.. 재시도...
  [2022-11-29 01:06:11.592] [d-2XXOO3JFK][stdout]> Health Check 성공
  [2022-11-29 01:06:11.621] [d-2XXOO3JFK][stdout]> 전환할 Port: 8082
  [2022-11-29 01:06:11.621] [d-2XXOO3JFK][stdout]> Port 전환
  [2022-11-29 01:06:11.632] [d-2XXOO3JFK][stdout]set $service_url http://127.0.0.1:8082;
  [2022-11-29 01:06:11.634] [d-2XXOO3JFK][stdout]> nginx Reload
  [2022-11-29 01:06:11.654] [d-2XXOO3JFK][stderr]Redirecting to /bin/systemctl reload nginx.service
  ```
- 한 번 더 배포 후 `ps -ef | grep java`로 real1, 2 profile 모두 실행 중인 것 확인
  ```
  [ec2-user@ip-172-31-41-214 deployment-logs]$ ps -ef | grep java
  ec2-user  9649     1  0 Nov28 ?        00:00:22 java -jar -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties -Dspring.profiles.active=real /home/ec2-user/app/step2/springaws-1.0.1-SNAPSHOT.jar
  ec2-user 13994     1  2 00:57 ?        00:00:16 java -jar -Dspring.config.location=classpath:/application.properties,classpath:/application-real1.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties -Dspring.profiles.active=real1 /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128155625.jar
  ec2-user 19129     1  9 01:05 ?        00:00:15 java -jar -Dspring.config.location=classpath:/application.properties,classpath:/application-real2.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties -Dspring.profiles.active=real2 /home/ec2-user/app/step3/springaws-1.0.1-SNAPSHOT-20221128160441.jar
  ec2-user 21480 29230  0 01:08 pts/0    00:00:00 grep --color=auto java
  ```
  - 현재 nginx가 보는 포트는 8082인 상태
