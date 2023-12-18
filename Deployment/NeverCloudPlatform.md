# 1. 개요

- 공모전 기준으로 Never Cloud Platfrom을 이용한 배포의 자세한 내용을 기록하기 위함.
- Docker, Nginx, Jenkins 사용 (쿠버네티스 등 그 외 사용 x)

# 2. NCP 배포 서버 구매

## 2-1. 서버 구매

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/1449b1ad-a06d-4a20-abfd-e93ada58fd8e/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/e47a8fec-6025-48b2-a686-80f56651fd8f/Untitled.png)

- 구매시 참고 사항
  - Micro는 무료 서버임. 그러나 빌드 시 30분 이상 소요되므로 추천하지 않음.
  - 서버 테스트 용으로는 Compact 이상이면 충분함.
  - 개인적 의견으로 Compact도 좀 오래 걸리긴 함….

## 2-2. AGC 설정

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/193b75e2-7343-45dd-b671-c408b3d4315e/Untitled.png)

- AGC 는 외부에서 서버로 접근을 허용해 주는 규칙을 정하는 것임(한마디로 방화벽!)
  - Load Balencer를 사용할 경우 여기에 적용해주면 됨.
    (이번 프로젝트에서는 사용하지 않았으므로 생략함)
- 현재 프로젝트 기준으로 아래 참고.
  | 포트 | 설명 |
  | ----- | -------------- |
  | 22 | SSH |
  | 80 | HTTP |
  | 443 | HTTPS |
  | 3000 | React |
  | 5012 | MySQL |
  | 27017 | MongoDB |
  | 8761 | Spring Eureka |
  | 8001 | Spring Gateway |
  | 64000 | member-service |
  | 65000 | map-service |
  | 9090 | Jenkins |
  | 6379 | Redis |
  - 22 : pem 키를 통해 서버에 접속하여 사용하기 위함.
  - 80, 443 : 각각 http, https 접근을 허용하기 위함.
  - 3000 : Front-End 표시
  - 5012, 27017, 6379 : DB에 접근하기 위한 포트
  - 64000, 65000 : Swagger 이용을 위함.
  - 8001 : MSA 에서 접속 시 해당 기능으로 연결해 주기 위함.
  - 8761 : MSA 에서 각 기능간의 통신을 위해 구별해주기 위함.
  - 9090 : Jenkins 접속을 위한 포트
  - 0.0.0.0/0 은 모든 IP에서 접근하겠다(알아서 설정)
  - +) 8080포트도 열어야 하는듯 (확인 필요)

## 2-3. 포트 포워딩

### 1. 공인 IP 생성 및 등록

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/150a5c70-912b-4edc-a213-32698a9fef2f/Untitled.png)

### 2. 포트 포워딩 등록

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/5c875bd2-5d40-401e-840c-059a10b29efc/Untitled.png)

- AGC로 연결되는 포트를 설정해 주었으니 설정은 생략함.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/69c295e6-d5e3-45f5-bb95-19c568e0e2d3/Untitled.png)

- 의미 : 서버 접속용 IP의 8080 포트로 들어왔을 때, 내부 IP의 22번 포트로 SSH 확인 후 들어보내주겠다는 소리!

- **공인 IP vs 서버 접속용 공인 IP**
  공인 IP는 외부에서 접속하는 IP, 즉 도메인 주소와 같은 역할임.
  서버 접속용 공인 IP는 서버를 접속하기 위한 IP로 외부 접근을 한번 더 차단시키기 위한 프록시 서버라고 생각하면 됨
  즉, 서버 접속용 IP로 들어가게 되면 내부에서 SSH가 맞는지 인증을 거치게 되고 서버로 연결 해주겠다는 의미임.

# 3. 서버 설정

- 모든 설명은 **mobaxterm** 프로그램을 이용하여 설명함.

## 3-1. 서버 접속

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/ad6d479c-6c34-4bea-b054-bfed674b0030/Untitled.png)

- 서버 접속용 공인 IP, 포트, pem키 등록

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/b3ffa99f-ffab-423f-b982-8ed5b6764eef/Untitled.png)

- 서버 접속 시 필요한 비밀번호는 사이트 내 관리자 비밀 번호에서 확인

## 3-2. 방화벽 설정

```bash
# 방화벽 확인
sudo ufw status

# 방화벽 허용
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https


# 만약 sudo ufw status를 했는데 inactive가 뜬다면?
sudo ufw enable
```

## 3-3. 자바 다운 및 초기 설정

### 1. 패키지 최신 버전으로 변경

```bash
sudo apt-get update
```

```bash
sudo apt-get upgrade
```

### 2. 자바 설치

```bash
# 설치 가능한 리스트
sudo apt-cache search openjdk
```

```bash
# 설치 - 이번 프로젝트에서는 11을 사용했으므로 11 설치
sudo apt-get install openjdk-11-jdk
sudo apt-get install openjdk-11-jre
```

```bash
# 설치 확인
java --version
javac --version
```

### 3. 자바 환경변수 설정

```bash
# 파일 편집
sudo vim .bashrc
```

```bash
# .bashrc 파일에서 마지막에 추가
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
export PATH=$PATH:$JAVA_HOME/bin
```

```bash
# 반영하기
source .bashrc
```

```bash
# 확인하기
echo $JAVA_HOME
```

- vim 명령어
  i : 문서 편집 시 사용
  esc : 편집 완료 시 나가기
  :w : 저장하기
  :qa : 나가기

## 3-4. DB 설정

### 1. MySQL

```bash
# 설치
sudo apt-get install mysql-server
```

```bash
# root 계정 접속
mysql -u root -p

# User 생성 이떄 %는 어느 호스트에서나 접속 가능하다는 와일드 카드
create user 'username'@'%' identified by 'password';

# 계정 확인
use mysql;
select user, host from user;
```

```bash
# DB 생성
create database dbname;
use dbname;

# DB 권한 부여
grant all privileges on dbname.* to 'username'@'%';
flush privileges; // DB에 변경사항 있음을 알림

# DB 나가기
exit;
```

```bash
# 외부 접속 허용하기
cd /etc/mysql/mysql.conf.d
sudo vim mysqld.cnf
```

```bash
# bind-address 를 127.0.0.1-> 0.0.0.0 변경
# port 를 5012로 변경

```

```bash
# mysql 재시작
systemctl restart mysql.service

# 서버 재시작 시 자동 활성화
sudo systemctl enable mysql

# 변경된 포트번호 확인 -> mysql이 listen 중이면 정상
sudo netstat -tulpn | grep 5012
sudo lsof -i :5012

# mysql 실행 중 확인
sudo systemctl status mysql
sudo ufw allow 5012
```

```bash
# 원격 DB 접속
hostname: 네이버 클라우드 공인 IP
port: 5012
username: 위에서 설정한 admin
password: 위에서 설정한 비밀번호
```

### 2. MongoDB

```bash
# MongoDB 설치
sudo apt-get install mongodb

# 시작
sudo systemctl start mongodb
```

```bash
# 설정 변경
# bindip를 0.0.0.0으로 변경
sudo vim /etc/mongodb.conf

# mongodb 다시 시작
sudo systemctl stop mongodb
sudo systemctl start mongodb

# 재시작시 자동 실행
sudo systemctl enable mongodb
```

```bash
mongo

# db 설정
use admin

# user가 pwd 를 가지며, 해당 db에 읽기 쓰기 권한을 가짐
db.createUser({
  user: "username",
  pwd: "password",
  roles: [
    {
      role: "readWrite",
      db: "dbname"
    }
  ]}
)

exit
```

```bash
# 확인
sudo systemctl status mongodb

# 포트 확인
sudo netstat -tulpn | grep 27017
sudo lsof -i :27017
sudo ufw allow 27017
```

```bash
# mongodb 외부 연결
hostname: 네이버 클라우드 공인 IP
port: 27017
username: 위에서 설정한 admin
password: 위에서 설정한 비밀번호
```

### 3. Redis

```bash
# 설치
apt install redis-server

# ncp가 Ipv6 지원안함에 따라 수정 필요
sudo vim /etc/redis/redis.conf

# bind 127.0.0.1 -> 0.0.0.0
bind 0.0.0.0 로 변경

# 추가
maxmemory 1g
maxmemory-policy allkeys-lru
requirepass password
```

```bash
# 재설치 후 초기화
apt install redis-server
sudo service redis-server restart
```

```bash
# 상태 확인
sudo systemctl status redis-server

# 포트번호 확인
netstat -nlpt | grep 6379

# 서버 재시작 시 자동 재실행 하도록 설정
sudo systemctl enable redis-server.service
sudo ufw allow 6379
```

```bash
# 원격 접속 시
hostname: 네이버 클라우드 공인 IP
port: 6379
username:
password: 위에서 requirepass에 설정한 비밀번호
```

# 4. Docker 설치

## 4-1. 서버 구조

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/c16fea96-40f6-4c77-ba41-ceb76d37bb7f/Untitled.png)

- Docker 내부에 설치할 것 : Jenkins, Front-End, Back-End 기능 각각
- Docker 설치 → Jenkins 설정 → Docker 설정 → Nginx 순서대로 진행할 예정
- 서버/각 폴더마다 Dockerfile, Docker-Compose.yml, deploy.sh 파일을 만들 것임.

## 4-2. 서버에 Docker 설치

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo apt install docker-compose

#정상 설치 되었는지 확인
sudo docker -v
sudo docker compose version
```

# 5. Jenkins 설치

- Jenkins는 자동 배포를 위해 사용한다.
- 간단한 순서는 아래와 같음
  - 1. github에 접근해서 프로젝트를 clone해 온다
  - 2. clone해 온 프로젝트를 빌드한다.
  - 3. 빌드된 파일에서 배포를 위한 파일(back의 경우 .jar 파일)을 서버에 복사해 온다
  - 4. .jar 파일을 실행시켜서 배포한다.

## 5-1. 서버 docker 설정

- 여러 방식이 존재하지만 이번 프로젝트에서는 Jenkins는 Docker 내부에 설치함.

```bash
# docker compose 설정
sudo vim docker-compose.yml
```

```bash
version: '3'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /jenkins:/var/jenkins_home
    ports:
      - "9090:8080"
    user: root
    cap_add:
      - SYS_ADMIN
    privileged: true

# 여기서 부턴 추가
# 프로젝트에 하지는 않았으나 아래와 같이 할 경우 DB도 컨테이너에 넣어서 관리할 수 있는것 같음

mysql:
    image: mysql:latest
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD:
      MYSQL_DATABASE: dbname
      MYSQL_USER: username
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"

  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx-config:/etc/nginx/conf.d
    depends_on:
      - my-app

  mongodb:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017"

  redis:
    image: redis:latest
    container_name: redis
    ports:
      - "6379:6379"

```

```bash
# 업로드
sudo docker-compose up -d
```

```bash
# container 실행 확인
sudo docker ps
```

## 5-2. Jenkins Docker 내부 설정

```bash
# container 내부에 Docker 설치
# compose도 같이 설치해서 사용하기 위함.

# 컨테이너 내부 접속
sudo docker exec -it jenkins /bin/bash
```

```bash
# Docker 설치
apt-get update
apt-get install ca-certificates curl gnupg lsb-release
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update

apt-get install docker-ce docker-ce-cli [containerd.io](http://containerd.io/) docker-compose-plugin docker-compose
```

## 5-3. Jenkins 초기 설정

- 접속 주소 : http://공인 IP:9090
- 비밀 번호 확인

```bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/d4109238-4099-493c-bf0b-46b685f1020f/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/a3ca23bb-36b7-4394-866d-2f95ccfd5843/Untitled.png)

```bash
# 플러그인 설치 목록
NodeJs -> 프론트 자동 배포시 필요
GitHub Integration -> 깃헙 연결하기 위해 필요
ssh agent -> deployment
```

- credential 설정
  - github에 접근하기 위함.
  - setting → Developer Setting → Personal access tokens → Tokens
    (github)
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/1db88c7a-5b1e-47b4-b238-383d87206697/Untitled.png)
    (jenkins)
  - jenkins관리 → System
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/c8854896-7ed0-45e0-818f-f4d27d15fcfb/Untitled.png)
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/7b11d96e-43ea-433d-ba6a-de54c20a7ff2/Untitled.png)

## 5-4. Jenkins Item 설정

- 새로운 item → 이름, pipeline

```bash
# 현재 프로젝트에서 사용할 item
item1 : goodnews-discovery
item2 : goodnews-gateway
item3 : goodnews-map
item4 : goodnews-member
item5 : goodnews-fe
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/ad116bbf-eae4-4353-83af-3d813b1187e8/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/88d902bc-5620-4ce7-a869-90c2b0ed67b2/Untitled.png)

- item script

  - 참고해야 할 점
    - url은 clone 해 올 github repository
    - git clone에 있는 credentialsId는 github에서 발급해온 토큰이 들어 있는 곳
    - ssh_key는 server에 접속하기 위한 key
    - docker내부 → 서버 접속 시 접근을 위한 키 발급 과정이 필요함 (할 예정)
    - 사용되는 ip는 서버 접속용 공인 IP임
    - 코드 설명(스크립트 내용)
      deployment의 sh 내부 코드는 jenkins에서 접속해서 server의 터미널 창에 알아서 써주겠다는 의미임
      즉, 내가 직접 쳐도 똑같은 일을 해준다는 것(직접 치면 수동 배포임)
      **ssh** → 해당 정보로 접속하겠다. 이때 접속된 곳은 docker내부의 jenkins로 연결됨
      **scp** → 앞의 경로(jenkins 도커 내부 경로) 에 있는 파일을 뒤의 경로(서버 경로)로 복붙 해주겠다!
      **ssh** → 서버로 들어가 [deploy.sh](http://deploy.sh) 를 실행시키겠다
      (deploy.sh는 jar 파일을 실행하고 배포되는 것을 자동으로 실행시키기 위한 명령어 모음(이후 절차에서 작성 예정))
  - goodnews-discovery

  ```bash
  pipeline {
      agent any

      stages {
          stage('Git clone') {
              steps {
                  git branch: 'master', credentialsId: 'github_key', url: 'https://github.com/Kim-Yejinn/goodnews.git'
              }
          }
          stage('Build') {
              steps {
                  dir("./BE-WEB/discovery") {
                      sh "chmod +x gradlew"
                      sh "./gradlew clean build"
                  }
              }
          }
          stage('Deployment') {
              steps {
                  sshagent(credentials: ['ssh_key']) {
                      sh '''
                          ssh -o StrictHostKeyChecking=no root@210.89.190.196 -p 8080
                          scp /var/jenkins_home/workspace/goodnews-discovery/BE-WEB/discovery/build/libs/discovery-0.0.1-SNAPSHOT.jar root@ncp-server-study:/root/goodnews-discovery/
                          ssh -p 8080 -t root@210.89.190.196 ./deploy-discovery.sh
                      '''
                  }
                  timeout(time: 30, unit: 'SECONDS') {
                  }
              }
          }
      }
  }
  ```

  - goodnews-gateway

  ```bash
  pipeline {
      agent any

      stages {
          stage('Git clone') {
              steps {
                  git branch: 'master', credentialsId: 'github_key', url: 'https://github.com/Kim-Yejinn/goodnews.git'
              }
          }
          stage('Build') {
              steps {
                  dir("./BE-WEB/gateway") {
                      sh "chmod +x gradlew"
                      sh "./gradlew clean build"
                  }
              }
          }
          stage('Deployment') {
              steps {
                  sshagent(credentials: ['ssh_key']) {
                      sh '''
                          ssh -o StrictHostKeyChecking=no root@210.89.190.196 -p 8080
                          scp /var/jenkins_home/workspace/goodnews-gateway/BE-WEB/gateway/build/libs/gateway-0.0.1-SNAPSHOT.jar root@ncp-server-study:/root/goodnews-gateway
                          ssh -p 8080 -t root@210.89.190.196 ./deploy-gateway.sh
                      '''
                  }
                  timeout(time: 30, unit: 'SECONDS') {
                  }
              }
          }
      }
  }
  ```

  - goodnews-map

  ```bash
  pipeline {
      agent any

      stages {
          stage('Git clone') {
              steps {
                  git branch: 'master', credentialsId: 'github_key', url: 'https://github.com/Kim-Yejinn/goodnews.git'
              }
          }
          stage('Build') {
              steps {
                  dir("./BE-WEB/map") {
                      sh "chmod +x gradlew"
                      sh "./gradlew clean build"

                  }
              }
          }
          stage('Deployment') {
              steps {
                  sshagent(credentials: ['ssh_key']) {
                      sh '''
                          ssh -o StrictHostKeyChecking=no root@210.89.190.196 -p 8080
                          scp /var/jenkins_home/workspace/goodnews-map/BE-WEB/map/build/libs/map-0.0.1-SNAPSHOT.jar root@ncp-server-study:/root/goodnews-map/
                          ssh -p 8080 -t root@210.89.190.196 ./deploy-map.sh
                      '''
                  }
                  timeout(time: 30, unit: 'SECONDS') {
                  }
              }
          }
      }
  }
  ```

  - goodnews-member

  ```bash
  pipeline {
      agent any

      stages {
          stage('Git clone') {
              steps {
                  git branch: 'master', credentialsId: 'github_key', url: 'https://github.com/Kim-Yejinn/goodnews.git'
              }
          }
          stage('Build') {
              steps {
                  dir("./BE-WEB/member") {
                      sh "chmod +x gradlew"
                      sh "./gradlew clean build"

                  }
              }
          }
          stage('Deployment') {
              steps {
                  sshagent(credentials: ['ssh_key']) {
                      sh '''
                          ssh -o StrictHostKeyChecking=no root@210.89.190.196 -p 8080
                          scp /var/jenkins_home/workspace/goodnews-member/BE-WEB/member/build/libs/member-0.0.1-SNAPSHOT.jar root@ncp-server-study:/root/goodnews-member
                          ssh -p 8080 -t root@210.89.190.196 ./deploy-member.sh
                      '''
                  }
                  timeout(time: 30, unit: 'SECONDS') {
                  }
              }
          }
      }
  }
  ```

  - goodnews-fe

  ```bash
  pipeline {
      agent any

      stages {
          stage('Git clone') {
              steps {
                  git branch: 'master', credentialsId: 'github_key', url: 'https://github.com/Kim-Yejinn/goodnews.git'
                  echo "Current workspace: ${workspace}"
              }
          }

          stage('Deployment') {
              steps {
                  sshagent(credentials: ['ssh_key']) {
                      sh '''
                          ssh -o StrictHostKeyChecking=no root@210.89.190.196 -p 8080 "rm -rf /root/goodnews-front/node_modules"
                          scp -r /var/jenkins_home/workspace/goodnews-fe/FE-WEB/good-news/* root@ncp-server-study:/root/goodnews-front
                          ssh -p 8080 -t root@210.89.190.196 ./deploy-front.sh
                      '''
                  }
              }
          }
      }
  }
  ```

## 5-5. Credential 설정(ssh설정)

```bash
# Jenkins Docker 접속
docker exec -it jenkins /bin/bash

# ssh 발급
# enter 누르면 기본 설정
ssh-keygen -t rsa

# 해당 내용을 복사해서 서버에 넣어줄것임
# .pub를 넣어 주어야 함!!!
cat root/.ssh/id_rsa.pub

ctrl + D 로 나가기
```

```bash
# (원격 서버)에 ssh 붙여 넣기 후 저장
vim .ssh/authorized_keys
```

```bash
sudo vim /etc/ssh/sshd_config

PubkeyAuthentication yes # 이거 주석 해제하기

systemctl restart ssh
```

## 5-6. ssh_key 등록

- enter directly에 직전에 만들었던 id_rsa 키 넣으면 됨(——어쩌고 —— 이 부분도 다 넣어야 함)
- 주의사항) authorized_keys에 들어가는 것은 public key(id_rsa.pub) 이고 이 부분에 들어가는 것은 private key(id_rsa) 이다.
- ssh 인증
  ssh 인증에 사용되는 key 는 public key와 private key의 pair로 이루어져 있어서 동일 여부를 판단한다.
  private key는 접속하려고 하는 곳에서 가지고 있는 key 이며, public key는 접속 될 서버에서 가지고 있는 key이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/6413b18a-f7b8-4a31-8a11-74c12ecf7224/Untitled.png)

# 6. Docker 파일 설정

- 파일 구조
  - 아래와 같이 각각의 파일에는 docker-compose.yml, Dockerfile이 있음
  - 또한, 상위 폴더에는 [deploy-xxx.sh](http://deploy-xxx.sh) 가 있는 구조
  - 각각의 디렉토리와 파일을 직접 만들면 되는 거임!
  - 여기서 docker-compose.yml, Dockerfile, deploy.sh만 각각 만들면 됨, 나머지 파일은 젠킨스를 이용해서 가져올것임

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/48f205c0-153c-49b8-bedf-e08b182515b2/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/63a1c587-3e53-483a-9897-b0e9e9ef3e69/Untitled.png)

- goodnews-discovery
  - docker-compose.yml
  ```bash
  version: '3'
  services:
    spring-goodnews-discovery:
      build:
        context: .
        dockerfile: Dockerfile
      image: goodnews-discovery
      container_name: goodnews-discovery
      ports:
        - "8761:8761"
  ```
  - Dockerfile
  ```bash
  FROM adoptopenjdk/openjdk11
  COPY ./discovery-0.0.1-SNAPSHOT.jar /discovery-0.0.1-SNAPSHOT.jar
  CMD ["java","-jar","discovery-0.0.1-SNAPSHOT.jar"]
  ```
  - deploy-discovery.sh
  ```bash
  cd goodnews-discovery
  sudo docker-compose down
  sudo docker-compose up -d --build
  y | sudo docker system prune
  ```
- goodnews-gateway
  - docker-compose.yml
  ```bash
  version: '3'
  services:
    goodnews-gateway:
      build:
        context: .
        dockerfile: Dockerfile
      image: goodnews-gateway
      container_name: goodnews-gateway
      ports:
        - "8001:8001"
  ```
  - Dockerfile
  ```bash
  FROM adoptopenjdk/openjdk11
  COPY ./gateway-0.0.1-SNAPSHOT.jar /gateway-0.0.1-SNAPSHOT.jar
  CMD ["java","-jar","gateway-0.0.1-SNAPSHOT.jar"]
  ```
  - deploy-gateway.sh
  ```bash
  cd goodnews-gateway
  sudo docker-compose down
  sudo docker-compose up -d --build
  y | sudo docker system prune
  ```
- goodnews-map
  - docker-compose.yml
  ```bash
  version: '3'
  services:
    goodnews-map:
      build:
        context: .
        dockerfile: Dockerfile
      image: goodnews-map
      container_name: goodnews-map
      ports:
        - "65000:65000"
  ```
  - Dockerfile
  ```bash
  FROM adoptopenjdk/openjdk11
  COPY ./map-0.0.1-SNAPSHOT.jar /map-0.0.1-SNAPSHOT.jar
  CMD ["java","-jar","map-0.0.1-SNAPSHOT.jar"]
  ```
  - deploy-map.sh
  ```bash
  cd goodnews-map
  sudo docker-compose down
  sudo docker-compose up -d --build
  y | sudo docker system prune
  ```
- goodnews-member
  - docker-compose.yml
  ```bash
  version: '3'
  services:
    goodnews-member:
      build:
        context: .
        dockerfile: Dockerfile
      image: goodnews-member
      container_name: goodnews-member
      ports:
        - "64000:64000"
  ```
  - Dockerfile
  ```bash
  FROM adoptopenjdk/openjdk11
  COPY ./member-0.0.1-SNAPSHOT.jar /member-0.0.1-SNAPSHOT.jar
  CMD ["java","-jar","member-0.0.1-SNAPSHOT.jar"]
  ```
  - deploy-member.sh
  ```bash
  cd goodnews-member
  sudo docker-compose down
  sudo docker-compose up -d --build
  y | sudo docker system prune
  ```
- goodnews-front

  - docker-compose.yml

  ```bash
  version: '3'
  services:
    goodnews-fornt:
      build:
        context: .
        dockerfile: Dockerfile
      image: goodnews-front
      container_name: goodnews-front
      ports:
        - "3000:3000"
  ```

  - Dockerfile

  ```bash
  FROM node:18.16.1

  WORKDIR /goodnews-front

  COPY package.json .

  RUN npm install

  COPY . .

  EXPOSE 3000

  CMD ["npm", "run", "start"]
  ```

  - deploy-front.sh

  ```bash
  cd goodnews-front
  sudo docker-compose down
  sudo docker-compose up -d --build
  y | sudo docker system prune
  ```

- .sh 파일 권한 허용

```bash
chmod +x deploy-discovery.sh
chmod +x deploy-gateway.sh
chmod +x deploy-member.sh
chmod +x deploy-map.sh
chmod +x deploy-front.sh
```

- Tip : 어디서 문제인지 모를 때 직접 sh 안의 명령어 치면서 확인하면 원인 찾기 쉬움
- 여기까지 하면 jenkins 테스트 가능. 이때 만약 host 인증을 할 수 없다고 뜰 경우 스크립트의 scp 구문에 -o StrictHostKeyChecking=no 를 추가하면 된다
  - 예상 원인) key로 한번 접근하고 그 정보를 저장하고 있다가 비교해서 그 뒤에는 편하게 접속할 수 있게 하는데 그 한번을 해주지 않았기 때문으로 보임.
    이것의 예시가 처음 서버 접속할 때에는 비밀번호까지 입력해야 했지만 이후에는 root라는 이름만 쳐도 접속이 되는 것
    아마 수동 배포 한번 했으면 기록이 남아 있어서 이 오류가 안 뜨는 것 일수도 있음.
    (정확한 원인 확인 필요)

# 7. Nginx 설정

## 7-1. 설치 및 실행

```bash
# nginx 설치

sudo apt install nginx

# ipv6 오류 발생시
# https://velog.io/@evelon/NCP%EC%97%90%EC%84%9C-Ubuntu%EC%97%90-nginx-%EC%84%A4%EC%B9%98-%EC%8B%9C-E-Sub-process-usrbindpkg-returned-an-error-code-1-%EC%98%A4%EB%A5%98%EA%B0%80-%EB%82%98%EB%8A%94-%EA%B2%BD%EC%9A%B0
# vim /etc/nginx/sites-enabled/default로 설정 들어가서 IPv6 해당하는 [::] 이 줄 주석 처리

sudo service nginx start
sudo service nginx status

# 아래 설정 후 restart
sudo systemctl restart nginx
```

```bash
# /etc/nginx/sites-available/

##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
	listen 80 default_server;
	# listen [::]:80 default_server;

	# SSL configuration
	#
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	#
	# Note: You should disable gzip for SSL traffic.
	# See: https://bugs.debian.org/773332
	#
	# Read up on ssl_ciphers to ensure a secure configuration.
	# See: https://bugs.debian.org/765782
	#
	# Self signed certs generated by the ssl-cert package
	# Don't use them in a production server!
	#
	# include snippets/snakeoil.conf;

	root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

# 여기에 도메인 주소를 넣고 실행시키면 된다.
	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

	# pass PHP scripts to FastCGI server
	#
	#location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
	#	fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	#}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {
	#	deny all;
	#}
}

# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#	listen 80;
#	listen [::]:80;
#
#	server_name example.com;
#
#	root /var/www/example.com;
#	index index.html;
#
#	location / {
#		try_files $uri $uri/ =404;
#	}
#}
```

## 7-2. 인증서 설치

1. let’s Encrypt 설치

```bash
sudo apt-get install letsencrypt
```

1. Certbot 설치

```bash
sudo apt-get install certbot python3-certbot-nginx
```

1. certbot 동작

```bash
sudo certbot --nginx
```

- 이메일 입력, 약관동의, 이메일 발송동의, 도메인 입력

- 기타 설정을 포함한 최종 nginx conf 파일( /etc/nginx/sites-available/default)

```bash
server {
    listen 80;
    server_name 서버주소1 서버주소2;

    if ($host = 서버주소) {
        return 301 https://www.$host$request_uri;
    }

    return 301 https://www.서버주소$request_uri;
}

server {
    listen 443 ssl;
    server_name 서버주소1 서버주소2;
    ssl_certificate /etc/letsencrypt/live/www.서버주소/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/www.서버주소/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:3000;
    }

    location /api {
        proxy_pass http://localhost:8001;
    }

    location /images {
        alias /home/upload;
        try_files $uri $uri/ =404;
    }
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    }
}
```

- ssl, 도메인 없이 테스트 코드

  ```bash
  server {
      listen 80 default_server;

      location / {
          proxy_pass http://localhost:3000;
      }

      location /api {
          proxy_pass http://localhost:8001;
      }

      location /images {
          alias /home/upload;
          try_files $uri $uri/ =404;
      }
  }
  ```

# 8. Github Webhook

- repository의 setting 에서 webhook 추가
- url은 “ jenkins Url/github-webhook/ “

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e528a434-c1c6-4e4e-a7e1-449220275f53/cfa4031e-82fc-443c-8e84-daec4447ce49/Untitled.png)
