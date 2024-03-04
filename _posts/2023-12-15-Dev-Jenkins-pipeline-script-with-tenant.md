---
layout: single
title: "multi-tenant 환경에서 GCP-Jenkins CI/CD 트러블슈팅 기록"
categories:
  - Dev
tags:
  - GCP
  - Jenkins
  - Multitenancy
  - Dockerfile
  - Jenkinsfile
  - SSO
  - locale
  - sed
---

# 작업 환경

- 사용 기술 : Jenkins, GCP(Artifact Registry, GKE, Ingress, )
- 테넌트가 2개(`tnt1`, `tnt2`)인데 개발 환경(`dev`, `stg`, `prd`) 별로 설정(보안 파일 등)이 달랐음
  - 테넌트는 Jenkins 빌드 페이지에서 Jenkins 파라미터와 함께 빌드 기능을 사용해서 선택함
  - Jenkins 파라미터로 테넌트 구분, `BRANCH_NAME`으로 개발 환경 구분
  - `dev` 환경은 두 테넌트가 통합해서 하나의 환경을 사용하는 구조였음
- 웹, 앱 서버 별로 적용해야 하는 도구들도 달랐음
  - 웹 : yarn 빌드, GKE 배포
  - 앱 : 튜나 적용, maven 빌드, 소나큐브 적용, GKE 배포
- 아래 내용들은 대부분 앱 서버와 관련된 설정임

# GCP 설정

- 개발 환경 별로 아예 프로젝트를 나눠서 진행했음
- 네트워크 보안은 하나의 프로젝트로 통합해서 관리함
  - 태그를 사용해서 target group 설정 가능
- 프로젝트 이름과 실제 인스턴스에 사용되는 주소가 다를 수 있기 때문에 유의해야 함

# SSO 구성 과정

1. `context-security.xml` 파일에서 `SAMLContextProvider` 클래스를 빈으로 등록할 때 Entity id가 필요한데, 기존에는 이 값을 `spring.profiles.active`가 포함된 주소로 등록해놓음

   - 기존 `application.yml`에서는 테넌트가 하나였기 때문에 `dev`, `stg`, `prd`로 프로파일을 구분했는데, `spring.profiles.active`로 현재 프로파일 이름을 참조하는 개발단 코드가 존재함
   - Jenkinsfile에서는 테넌트를 포함해서 구분한 내용을 바탕으로 빌드할 때 적용할 프로파일을 설정하기로 정함(`stg_tnt1` 등)
   - 우선 `dev`에서 기능 작동을 확인하기로 함

2. 환경이 하나로 통합되어 있는 `dev`에서 먼저 sso 연동을 진행하고 정상 작동 확인함

   - `sso-metadata.xml`, `context-security.yml` 내부 커스터마이징

3. `context-security.xml` 내부에서 프로파일 별로 entity id 적용 방법을 찾아봄

   - `appliation.yml` 내부 프로파일 이름 변경 후 `spring.profiles.active` 그대로 사용
   - `@Values()` 어노테이션 내부에서 스트링 조작
     - 개발단 코드를 수정해야 해서 힘듦
   - 태그 내부 삼항연산자(`...value="#{${spring.profiles.active}=='dev'...`)
   - if 태그(`<if test="${spring...`)
   - beans 태그(`<beans profile="dev"...`)
   - `sso-metadata.xml`과 같이 프로파일별로 파일 분리
   - `application.yml` 파일 내부에 환경 변수 추가 후 호출(`"${spring.sso.entity-id}"`)

4. 프로파일 이름을 변경했을 때 영향도를 분석했을 때 환경 변수를 따로 만드는게 낫다고 판단해서 `spring.sso.entity-id`로 환경 변수를 등록하고 `context-security.xml`에 적용함

# 소나큐브 적용할 때 인코딩 문제

- 소나큐브 적용 후 인코딩 문제로 빌드가 실패함
  ```jsx
  Failed to execute goal org.sonarsource.scanner.maven:sonar-maven-plugin:3.10.0.2594:sonar
  ...
  Malformed input or input contains unmappable characters:...
  ```
  - os locale과 소스 파일의 인코딩이 달라서 발생한 문제임
    ```jsx
    ...default locale : en_US, source code encoding : UTF-8...
    // 파일 이름에서 한글 깨짐
    ```
    - locale이 `en_US`이고 codeset이 생략되어 있었는데, 기본이 `POSIX`였기 때문에 한글이 지원되지 않았음
  - 아래 그림에서 codeset이 `POSIX`일 경우 한글 파일 이름이 제대로 표현되지 않지만, locale이 `en_US.UTF-8`일 경우 정상적으로 나오는 것을 확인할 수 있음  
    ![sonarqube-encoding1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/jenkins-pipeline-sonarqube-encoding1.png)
- 해결 방법
  - mvn 빌드 명령어 앞에 임시 환경변수 설정으로 locale 지정
    ![sonarqube-encoding2](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/jenkins-pipeline-sonarqube-encoding2.png)
- 시도한 방법들
  - 쉘 스크립트로 locale 직접 적용
    ```jsx
    apt-get install -y locales locales-all
    locale-gen en_US.UTF-8
    sed -i 's/\$/\\nLANG=en_US.UTF-8\\nLC_ALL=en_US.UTF-8/g' /etc/default/locale
    . /etc/default/locale
    ```
    - `sh ". /etc/default/locale"`이 적용되지 않아서 실패함(실행 후 locale을 실행해도 `POSIX` 그대로임)
      - Jenkins agent를 사용하는 환경이었는데, agent에 직접 액세스할 수가 없었기 때문에 터미널을 재시작할 수 없었음
  - sonarqube scanner에서 인코딩 옵션을 적용하는 방법도 존재함
- 시간이 여유롭다면 agenttemplate(이미지)에서 locale 설정을 미리 POSIX → en_US.UTF-8로 바꾸는게 더 효율적일 것 같음
