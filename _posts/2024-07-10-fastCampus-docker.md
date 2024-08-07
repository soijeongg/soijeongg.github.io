---
layout: post
read_time: true
show_date: true
title: "컨테이너 가상화 이해 "
date: 2024-07-10 15:00:20 -0600
description: "DevOps초격차 패키지 Online 파트1"
image: ../assets/img/uploads/docker.png
tags: 
    - coding
    - diary
    - nest
author: soi

toc: no # leave empty or erase for no TOC
---

### 컨테이너 기술
컨테이너 기술이란 애플리케이션을 언제든지 실행가능 하도록 모든 요소를 하나의 런타임으로 패키징한 공간이다.
하나로 묶어 경량의 격리한프로세스다.
운영체제 수준의 경량화를 제공한다.

- 도커가 Vm과 다른점
둘다 운영체제 수준의 가상화를 제공하지만 컨테이너의 운영체제는 커널이 없다. 
도커는 os위에 올라가는 플랫폼으로 os의 운영체제를 도커가 공유해서쓴다-> 커널은 도커에 포함되지 않는다. 
독립되서 다른 컨테이너에 연결하지 않는 휘발성 환경이다.
서버구성 os, 네트워크 , 개발도구 환경설정등 반복적인 작업에 낭비하지 않고 개발 그 차제에 집중할 수있다. 
도커 컨테이너는 이미지가 불변이기 때문에 항상 같은 컨테이너로 배포가 된다 -> 개발환경에서 테스트 거쳐 배포 할때 항상 같은 환경에서 실행할 수 있다. 

도커에서는 도커 런 하나로 여러 컨테이너를 동시 실행 가능하다. 

### 컨테이너는 어떤 타입으로 생성되나요?
컨테이너 패키징 메커니즘
컨테이너는 이미지를 패키징해서 run해서 사용할 수 있게 만들어준다. 
3가지로 나눠짐.
시스켐컨테이너, 애플리케이션 컨테이너, 라우터 컨테이너.

1. 시스템 컨테이너 
호스트 os위에 os를 깔아 배 -> 또다른 vm의 형태로 내부에 다양한 애플리케이션이나 라이브러리 도구를 설치, 실행한다.
대포적인 걸로 LXC(리눅스 컨테이너), openVD, Linux Vserver
초창기에 나왔던 버전으로 일반적인 Vm이 아닌 가상의 컨테이너 Vm이다.

2. 애플리케이션 컨테이너
애플리케이션 컨테이너가 도커의 주역할(다 되는데 도커를 쓰는 이유는 애플리케이션 컨테이너로 쓸려고)
단일 애플리케이션 실행을 위해 서비스를 패키징하고 실행하도록 설계된것이다.
보통 시스템의 PID 1번을 시스템D, 근데 시스템 컨테이너의 1번또한 os 주체가 1번.
근데 애플리케이션은 호스트를 떠나서 1번 PID자체가 os PID가 아닌 개별 애플리케이션이 1번.
3-tier 애플리케이션 같은 경우 각 tier(프론트-백-디비)를 개별 컨테이너로 실행해 연결.

### 도커란
여러계층의 애플리케이션을 분리하고 api를 사용해 연결한다. 
이미지를 통해 제공하고 퍼블릭이나 프라이빗하게 공유할 수 있다(도커허브의 프라이빗은 갯수가 1개밖에 안됨)
