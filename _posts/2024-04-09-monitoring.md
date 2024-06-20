---
layout: post
read_time: true
show_date: true
title: "도커로 그라파나, 프로메테우스 , node expoter를 적용해보자!"
date: 2024-04-09 15:00:20 -0600
description: "도커로 그라파나, 프로메테우스 , node expoter를 적용해보자!"

tags: 
    - coding
    - diary
    - hh99
author: soi

toc: no # leave empty or erase for no TOC
---

### 배경
전의 블로그 글을 보면 로컬에서 그라파나와 프로메테우스를 사용해서 모니터링을 해봤다 

여기에서는 도커가 아닌 로컬에서 실행하고 있었다 
그렇기 때문에 node expoter를 사용해 매트릭을 수집하는 것이 아니라 수동으로 직접 /metrics 엔드포인트로 값을 내보내고 있었다 

하지만 어차피 프로젝트 배포 과정에서 도커를 활용해 빌드를 할것이기 때문에 기본적으로 그라파나와 프로메테우스의 연동법을 공부했다면 로컬이 아니라 도커의 환경에서 사용하는것이 좋겠다고 생각했다 

### 기본 개념
도커는 이미지를 풀받아 사용한다 
예를 들어 node 프로젝트를 사용하고 싶으면 일단 from node:alpine 으로 적용한다 
그리고 이러한 도커 파일을 여거개 모아놓은것을 한꺼번에 사용하는것을 docker-compose라고 하고 이를 docker-compose.yml파일에 지정한다 

이 도커 컴포즈 파일또한 전과 마찬가지로 이미지를 풀받아 사용하거나 본인이 만든 도커 파일을 빌드해서 사용할 수 있다

- 본인의 파일을 빌드할경우 
```yml
 app:
    build: .
    ports:
      - "3000:3000"
 ```
 build에 도커파일의 path을 지정한다 
 지금 아무것도 없이 . 으로 한거는 dockerfile과 이 docker-compose 파일이 같은 수준에 있어서 찾아갈 수 있다는 의미이다 
 여기에는 없지만 dockerfile: 이라고 하면 도커 파일이 여러개 있어서 각각 도커 파일을 이름을 다르게 했을때 이 도커파일을 이름으로 지정하기 위해 사용한다 
 
 - 이미지를 pull받아 사용할 경우 
 ```yml
  prometheus:
    image: prom/prometheus
    ports:
      - "9091:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    depends_on:
      - node-exporter
 ```
prometheus로 이 컨테이너의 이름을 지정
그 후 어떤 이미지를 다운 받을껀지 지정해준다 이건 프로메테우스 여서  prom/prometheus라고 지정한다 
포트를 외부에서는 9091로 바인딩하고 9090으로 내부로 연결시킨다 
volumes안에는 프로메테우스의 설정파일을 넣어준다 
- ./prometheus.yml:/etc/prometheus/prometheus.yml 의 의미
voiumes안의 구문은 :앞과 뒤로 나뉜다 
:앞 -> 현재 도커컴포즈를 빌드할 도커컴포즈 파일이 있는 곳에서 프로메테우스 설정파일의 path
	  나는 전에 했던 프로메테우스 컴포즈 파일의 내용을 그대로 복붙해 따로 폴더를 만들지 않고 그냥 같은 수준에 넣어줬다 
만약 폴더를 만들고 안에 넣어 준다고 하면 /prometheus/prometheus.yml 이렇게 하면 될듯하다 
```yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "stocking"
    static_configs:
      - targets: ["stocking:3000"]
  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]
```
여기 보면 scrape_configs라고 있는데 매트릭을 수집할 엔드포인드를 지정한다 
일단 첫번쨰는 원래 내가 하던 엔드포인트를 보기 위해 넣어준거 
두번째가 진짜 node expoter를 사용해서 수집하는 데이터를 보기 위한 엔드포인트이다 
targets: ["node-exporter:9100"]인 이유는 로컬이 아니라 node-exporter라는 이름으로 도커컴포즈에서 빌드했기때문에 이걸 여기에서 사용한다고 지정해준거다 

### 도커 코드 
```yml
#부모 이미지 지정
FROM node:alpine
#도커 내부의 디렉토리를 생성
WORKDIR /app

# 외부 패키지 설치를 위해 package.json과 yarn.lock 파일 복사
COPY main/package.json .
COPY main/yarn.lock .

# 패키지 설치
RUN  yarn install

# 메인 폴더의 내용을 전부 복사 
COPY main .
# 현재 디렉터리에 있는 파일들을 이미지 내부 /app 디렉터리에 추가함

ADD     . /app
#로컬 빌드이기때문에 안의 .env파일이 있어 arg와 env로 사용할 필요 없음

RUN yarn prisma generate
CMD [ "node","src/app.js" ]
```

### 도커 컴포즈 코드 
도커 컴포즈 코드는 docker-compose.yml에 코드를 작성한다 
```yml
version: "3"
services:
  stocking:
    build: .
    ports:
      - "3000:3000"
  node-exporter:
    image: prom/node-exporter
    ports:
      - "9101:9100"
  prometheus:
    image: prom/prometheus
    ports:
      - "9091:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    depends_on:
      - node-exporter
  grafana:
    image: grafana/grafana
    ports:
      - "3002:3001"
    volumes:
      - ./grafana.ini:/etc/grafana/grafana.ini
```
여기 보면 4가지 작업이 진행되고 있다 
1. stocking이라는 도커 파일을 빌드하고 있다 (빌드로 도커 파일을 지정하는데 .하나 같은 위치에 있는 도커 파일)
2. prom/node-exporter를 이미지로 받아 실행한다 
볼륨에는 위에 썻듯이 설정파일을 적용한다 포트는 내가 이미 로켈에서는 로컬 node expoter를 사용해서 바꿨다 
3. prom/prometheus를 이미지로 받아 실행한다 포트도 이미 로컬에서 실행하고 있는게 있어서 외부바인딩을 바꿨다 
4. grafana/grafana를 이미지로 받아 실행한다 볼륨 안의 실행파일은 전에 로컬로 실행해놓은 파일을 그대로 복붙했다 
그리고 포트 번호는 원래 3000이고 내가 설정파일에 3001으로 바꿨는데 로컬 실행이랑 다르게 할려고 3002로 외부 바인딩 해줬다 

이렇게 포트 번호 바꾸니까 계속 나오던 
`Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:9100 -> 0.0.0.0:0: listen tcp 0.0.0.0:9100: bind: address already in use` 
이 에러도 안나온다 
나는 그동안 도커도 ec2에서 사용하듯이 다른 포트를 사용하고 있어서 포트가 겹치는거는 상관 없을꺼라고 생각했는데 외부바인딩이 문제였다 
로컬과 도커를 연결시켜주는 외부바인딩은 로컬에서 들어가는 것이기 때문에 이 포트는 다른 것과 겹치면 안된다 

이렇게 되면 이제 로컬에서 9091포트로 들어가면 도커 내부에서 9090포트로 바꿔서 들어갈 수 있게 된다 
![](https://velog.velcdn.com/images/soijeongg/post/6ba6000f-88df-42b5-8689-39f014327dea/image.png)
만약 이게 unknown이거나 down이면도커 데스크톱 들어가서 잘 되고있는지 확인하기!
아마 오류나서 실행이 안되고 있을것-> 이걸 제대로 바꿔줘야 실행이 된다 
혹시 이 node expoter는 되고 있는데 다른게 안되서 에러가 날 수도? 나는 그랬던거 같은데 일단 도커 데스크톱 들어가서 확인해보자 
모든 에러를 다 잡으면 이렇게 up이라고 뜬다 
그리고 원래 다른 엔드 포인트들은 여기 들어가면 어떤걸 수집하고 있는지 나오지만 애는 node-expoter가 있는 곳으로 열어야 뭘 수집하고 있는지가 뜬다
![](https://velog.velcdn.com/images/soijeongg/post/c865e971-fd91-44ed-a1d9-c9c424c8dcf7/image.png)
여기 들어가면 볼 수 있다 

### 그라파나 연동
그라파나를 위에 적용한 3032포트로 접속한다 
그러면 그동안 로컬에서 실행한 그라파나와 달리 도커의 그라파나에 들어가게 된다 
맨 처음 아이디와 비밀번호는 admin/admin 이다 
들어가면 커넥션에 들어가 데이터 소스에 들어간다 
그곳에 가 프로메테우스를 선택하고 주소를 node-expoter:9100을 사용한다 
node-expoter는 도커컴포즈 빌드할때 사용했던 이름 그리고 포트가 왜 외부바인딩이 아니라 내부 바인딩을 사용하냐면 외부에서 사용하는게 아니라 내부에서 사용하는거라고  생각한다 
연동이 잘되면 (test& save 인가 그 버튼 누르고 연동 되면 초록색으로 알려준다)
![](https://velog.velcdn.com/images/soijeongg/post/b32ffd0a-68ae-479c-a19a-209eb2de1b1f/image.png)

이렇게 잘 나온다 