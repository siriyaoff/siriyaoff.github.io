---
layout: single
title: "How to adapt Jenkins on EC2"
categories:
  - Dev
tags:
  - AWS
  - EC2
  - Jenkins
  - CI/CD
  - pm2
  - Git
  - Linux
---

# 1. Install Java, Jenkins to EC2

---

```bash
# remove other versions of java
yum list installed | grep java
yum remove java-1.8.0-openjdk*

# install corretto, jdk 11
sudo curl -L https://corretto.aws/downloads/latest/amazon-corretto-11-x64-linux-jdk.rpm -o jdk11.rpm
sudo yum localinstall jdk11.rpm
sudo /usr/sbin/alternatives --config java
java --version
rm -rf jdk11.rpm

# install jenkins
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
sudo service jenkins start
```

- corretto : aws에서 제공하는, openjdk의 multi-platform distribution

# 2. Jenkins configuration

---

## Port 변경

```bash
sudo vi /etc/sysconfig/jenkins
sudo vi /usr/lib/systemd/system/jenkins.service
```

- 둘 다 포트 `8080` → `8090`으로 변경(필요할 경우)

## 초기 설정

1. 브라우저에서 `8090` 포트로 들어가면 `/var/lib/jenkins/secrets/initialAdminPassword`를 입력하라는 창이 뜸
   - 그대로 복사해서 입력
2. Install suggested plugins로 설치
3. 계정 설정
4. Dashboard → Jenkins 관리 → 플러그인 관리 → Available plugins에서 github integration 설치(Install without restart)

## Github 연동

1. 깃헙 프로필 Settings → Developer settings → Personal access tokens → Tokens → Generate new token

   - Select scopes : `repo`, `admin:repo_hook`

   ```bash
   GITHUB_ACCESS_TOKEN
   ```

2. Jenkins 설정 페이지 → Jenkins 관리 → 시스템 설정 → GitHub → Add GitHub Server
   - Credentials - Add : Jenkins
     - Domain : Global credentials (unresetricted)
     - Kind : Secret text
     - Scope : Global
     - Secret : `{github token}`
     - ID : Secrettext
     - Description : Secret text
   - Test connection 했을 때 `Credentials verified~`가 나오면 됨

# 3. Jenkins WebHook

---

## 프로젝트 생성

Dashboard → new Item → Freestyle project

- General
  - Github Project : `{repo http address}`
- 소스 코드 관리 : Git
  - Repositories
    - URL : `{repo http address}`
    - Credentials → Add
      - Kind : Username with password
      - Scope : Global
      - Username : `{github ID}`
      - Password : `{github token}`
      - ID : usernamewithpassword
      - Description : usernamewithpassword
- 빌드 유발 : GitHub hook trigger for GITScm polling
- Build Steps - Execute shell
  ```groovy
  #!/bin/bash
  cd /var/lib/jenkins
  source .bashrc
  cd ./workspace/TomatoSaas
  npm install
  PM2HOME=/etc/pm2daemon pm2 start index.js
  ```
  - jenkins 웹페이지에서 홈 디렉토리를 바꿔도 적용이 잘 되지 않아서 명시해줌
  - github에서 요청이 들어와서 폴링한 클론은 `/var/lib/jenkins/workspace/`에 저장됨

## 파일 권한 변경

- 프로젝트를 홈 디렉토리에 받았을 경우
  ```bash
  sudo chmod 755 /home
  sudo chown ubuntu:ubuntu /home/{username} -R
  sudo chmod 700 /home/{username} /home/{username}/.ssh
  sudo chmod 600 /home/{username}/.ssh/authorized_keys
  ```
  - `home`, `.ssh`, `authorized-keys`는 권한이 변경되면 안됨
  - 기본 설정대로 했을 경우 `/var/lib/jenkins/workspace`에 설치되기 때문에 할 필요가 없음

## 깃 레포 설정

- Settings → Webhooks → Add webhook
  - Payload URL : `{eip}:{port}/github-webhook`
    - 302 error : 맨 뒤에 / 넣어주기
  - Content type : application/json

## Troubleshooting

### `No valid crumb~` 에러

**문제점**

- ec2의 eip로 요청을 보냈기 때문에 jenkins 서버에서 받을 때는 ip가 바뀌어서 crumb가 유효성을 상실하게 됨
  - crumb : username, sessionID, IP, salt를 포함하는 쿠키
    - salt : random data

**해결 방법**

- jenkins 홈 디렉토리에서 설정파일을 만들어줘야 함

  - `/var/lib/jenkins/init.groovy.d/myconf.groovy`

    ```groovy
    import hudson.security.csrf.DefaultCrumbIssuer
    import jenkins.model.Jenkins

    if(!Jenkins.instance.isQuietingDown()){
      if(Jenkins.instance.getCrumbIssuer()!=null){
        def instance = Jenkins.instance
        instance.setCrumbIssuer(new DefaultCrumbIssuer(false))
        instance.save()
        println 'excludeClientIPfromCrumb set: false'
      }else{
    	  println 'Nothing changed'
    	}
    }
    ```

- 서비스 재시작(`sudo service jenkins restart`)
- 에러난 payload 들어가서 Redeliver 후 응답 정상적으로 오는 것 확인

### `npm/pm2: command not found`

**문제점**

- `jenkins` 계정이 시스템 계정으로 생성되었을 때 문제가 생김
- Build step - Execute shell은 `jenkins` 계정으로 실행되는데, 이때 ec2-user과 같은 다른 계정에서 설치한 프로그램들은 공유가 되지 않기 때문에 찾을 수가 없음

**해결 방법**

- jenkins 계정을 user 계정으로 만들고 필요한 프로그램들을 설치해야 함
  - 시스템 계정은 권한이 부족해서 쉘을 `/bin/bash`로 바꿔도 설치할 수가 없음
- `/var/lib/jenkins` 등의 jenkins 설정 파일도 권한 바꿔줘야 함

**이미 시스템 계정으로 만들어졌다면?**

- 사용자 계정으로 재생성 후 경로를 적절히 바꿔주면 됨

```bash
sudo userdel jenkins
sudo useradd jenkins
```

- `userdel`으로 아무런 옵션을 주지 않고 계정을 지우면 사용자의 파일은 그대로 남아있음
- home directory를 `/var/lib/jenkins`로 바꾸고 싶을 경우

  1. `sudo vi /etc/passwd`에서 `jenkins` 계정의 home directory를 `/var/lib/jenkins`로 변경
  2. `/var/lib/jenkins/.bashrc`를 빈 파일로 만듦
  3. 홈 디렉토리에서 `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash`를 통해 nvm 설치 후, 나오는 명령어들을 `/var/lib/jenkins/.bashrc`에 붙여넣음

     ```bash
     export NVM_DIR="$HOME/.nvm"
     [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
     [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
     ```

  4. 터미널 재시작 후 `pm2 status`로 정상 작동 확인

- home directory를 `/home/jenkins`로 유지하고 싶을 경우
  1. 홈 디렉토리에서 nvm, node, pm2 설치 후 위 `NVM_DIR` 관련 명령어들 실행

# 4. pm2 daemon 공유

---

- 3까지 설정했을 때는 `ec2-user`, `jenkins` 두 사용자 계정에서 각각 pm2가 실행됨
  - nginx에서는 `ec2-user`의 pm2 daemon만 호스팅해줌(권한 문제 때문에 `jenkins` 계정의 pm2 데몬에서 index.js가 제대로 시작되지 않은 것일 수도 있음)
  - 두 곳에서 같은 포트를 사용하게 될 경우 충돌이 일어나기 때문에 두 계정에서 공유할 수 있는 pm2 daemon을 만드는게 나음

```bash
sudo groupadd pm2
sudo usermod -a -G pm2 jenkins
sudo usermod -a -G pm2 ec2-user

cd /etc
mkdir pm2daemon
PM2_HOME=/etc/pm2daemon pm2 start /var/lib/jenkins/workspace/TomatoSaas/index.js
```

- 여기까지 실행했을 때 `/etc/pm2daemon`에 pm2 daemon 관련 파일들이 생성된 것 확인

```bash
sudo chgrp -R pm2 /etc/pm2daemon
sudo chmod -R 770 /etc/pm2daemon
sudo chmod g+s /etc/pm2daemon
```

- 이후 그룹 권한 설정을 해주면 `jenkins`, `ec2-user` 모두 `/etc/pm2daemon`에 접근 가능

## Jenkins 삭제

- 서비스 중지
  ```bash
  service jenkins stop
  /etc/init.d/jenkins stop
  ```
- 파일 제거
  ```bash
  sudo yum remove jenkins
  rm /etc/init.d/jenkins
  rm -rf /var/lib/jenkins
  rm /etc/yum.repos.d/jenkins.repo
  ```
