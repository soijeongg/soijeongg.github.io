---
layout: post
read_time: true
show_date: true
title: "[nest.js error] ERR! code ENOENT 처리"
date: 2024-05-28 15:00:20 -0600
description: "[nest.js error] ERR! code ENOENT 처리"
image: https://i.pinimg.com/564x/af/68/60/af6860519c5c7ef577817bb41813d535.jpg
tags: 
    - coding
    - diary
    - hh99
author: soi

toc: no # leave empty or erase for no TOC
---
### 문제상황
갑자기 nest build를 할려고 하는데 갑자기 에러가 났다 
```bash
npm ERR! code ENOENT
npm ERR! syscall spawn sh
npm ERR! path /Users/jeongsoi/Desktop/tupeScript/Devcamp_nest/devcamp/node_modules/argon2
npm ERR! errno -2
npm ERR! enoent spawn sh ENOENT
npm ERR! enoent This is related to npm not being able to find a file.
npm ERR! enoent 
```
빨간색으로 나와서 정말 무서웠음...
찾아보니까 뭐 package.json을 삭제해라 해서 다시 해도 안되고 npm install할려고 해도 에러가 나서 진짜 망했다라고 생각했다 

### 진행
gpt한테 물어보니까 /bin/sh 경로에 sh 셸이 있어야 한다고 `ls -l /bin/sh`을 입력하라고 했는데 갑자기 ls를 못찾았다
```bash
zsh: command not found: vi
(base) jeongsoi@jeongsoiui-iMac ~ % ls -l /bin/sh

zsh: command not found: ls
```
이렇게 나오는데 뭔가 이상해서 찾아보니까 환경변수가 잘못되어 있으면 이렇게 나온다고 한다 

### 해결
먼저 임시로 `export PATH=%PATH:/bin:/usr/local/bin:/usr/bin` 입력
그러면 vi, ls등 기본 명령어를 사용할 수 있게 된다 
 `vi ~/.zshrc`를 입력해 편집하고 이상한 환경변수 삭제하고 나와서 source ~/.zshrc를 입력해 적용한다 
 잘됐는지 확인은 그냥 ls가 되는지 안되는지 해보면 됨
 제대로 나오면 잘된거