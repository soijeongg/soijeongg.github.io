---
layout: post
read_time: true
show_date: true
title: "ubuntu vm에서 docker-compose 돌리기"
date: 2025-01-14 10:00:20 -0600
description: "로컬에서 이미지 빌드 후 vm에 전송하기"
image:  https://image.yes24.com/goods/78586788/XL
tags: 
    - coding
    - diary
   
author: soi

toc: no

---
### 1. vm 접속후 도커 설치하기 위해 필요한 패키지 설치하기 위해 sudo 권한 갖기 

``` bash
su
```

입력 후 비밀번호 입력하라는 말이 나오면 기존에 접속할때 사용했던 비밀번호 입력한다 -> 유저 이름 뒤에 #이 붙으면 성공

### 2. 도커 설치

#### 2-1. 도커 설치하기 위해 필요한 패키지 다운로드

``` bash
sudo apt-get update
sudo apt-get install \
 apt-transport-https \
 ca-certificates \
 curl \
 software-properties-common
```
apt-transport-https: HTTPS 프로토콜을 사용하여 패키지를 다운로드할 수 있도록 지원하는 패키지
ca-certificates:HTTPS 통신 시 인증서 검증하는 데 필요한 인증서 묶음, 인증서가 유효한지 확인함
curl: URL에서 데이터 가져오거나 서버로 데이터 요청(api 요청)
software-properties-common: 우분투에서 ppa를 사용하기 위한 패키지

**PPA란**

Personal Package Archive로 우분투는 우분투 소프트웨어 센터에 등록된 프로그램만 다운 받을 수 있는데 안전하지만 이는 등록되지 않은 프로그램을 다운받을 수 없고 업데이트가 느리다는 단점이 있다 

이때 등록되지 않은 프로그램과 최신버전을 다운받을 수 있게 해주는 것이 PPA이다

#### 2-2. 도커 공식 GPG 키 추가 

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

GPG란 데이터를 암호화하고 서명하는데 사용되는 공개키 암호화 기술로 리눅스 패키지 관리 툴이 프로그램 패키지가 유효한지 확인하기 위해 설치전 gpg키로 검증

#### 2-3. 도커 리포지토리 추가 
```bash
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
```

apt리포지토리에 도커가 추가 

#### 2-4. 도커 CE 설치 
```bash
sudo apt-get update
sudo apt-get install docker-ce
```
기업용 EE와 무료인 CE중 CE설치

#### 2-5. 도커 설치 확인
```bash
docker --version
```
도커의 버전이 나오면 설치 완료

#### 2-6. 사용자에게 도커 권한주기 

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
sudo newgrp docker
```

### 3. 사용자의 로컬에서 도커 파일 생성 및 도커 컴포즈 이용해 도커 이미지 빌드하기 

#### 3-1. 서버의 도커 파일만들기 

도커 파일을 작성하고 이 도커파일을 빌드해 도커 이미지를 만든다 이 도커파일에는 도커의 토대가 될 이미지나 실행할명령어를 기재한다 

해당 도커 파일의 명령어는 이 [링크] (https://velog.io/@soijeongg/Dockerfile-%EC%9D%B4%EB%AF%B8%EC%A7%80%EB%A7%8C%EB%93%A4%EA%B8%B0)에서 확인 가능하다 
해당 서버의 레포지토리에  dockerfile을 생성한다 

도커의 이미지에 필요한 env를 정리한다 이 env들이 해당 도커 파일을 사용해 도커 이미지 생성시 필요하다 
```bash
COOKIE_SECRET
NODE_ENV
PORT
CORS
```
이런 env들을 arg라는 