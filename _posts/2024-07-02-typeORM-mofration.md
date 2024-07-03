---
layout: post
read_time: true
show_date: true
title: "typeOR 마이그레이션
"
date: 2024-07-02 15:01:20 -0600
description: "typeORM연결방법 및 레이어드 패턴에 IRequest와 DTO" 
image: https://i.pinimg.com/564x/cc/97/49/cc97499cc69c7ca091bae1238e7c5d59.jpg
tags: 
    - coding
    - cs
author: soi

toc: no # leave empty or erase for no TOC
---
typeORM 공부를 위해 혼자 만들고 있던 게시판 프로젝트에서 엔티티만들때 오타가 난지 모르고 만들었다가 이번에 수정했더니 오류가 났다 
처음에는 tsc 하고 node/index.js 해도 거기서 멈춰있더니 겨우 오류가 나왔더 
``` shell
Data Source has been initialized!
TypeORM connection error:  QueryFailedError: column "subCategorisName" does not exist
    at PostgresQueryRunner.query 
```
지금보면 subCategorisName가 없다 
마이그레이션을 적용하지않고 실행시켜서 이에러가 난것

### 마이그레이션 방법
1.  npm install -D ts-node tsconfig-paths 설치 
2. datasource 파일에 마이그레이션 파일 지정

migrations: ['src/database/migrations/**/*.ts'],
  subscribers: [],

3.package.json에서 스크립트를 작성
 "typeorm" : "ts-node -r tsconfig-paths/register ./node_modules/typeorm/cli.js --dataSource src/dataSource.ts",
    "migration:create": "ts-node -r tsconfig-paths/register ./node_modules/typeorm/cli.js migration:create ./src/database/migrations/Migration", //마이그레이션 생성
    "migration:generate": "npm run typeorm migration:generate ./src/database/migrations/Migration", //마이그레이션 생성(자동생성)-> 폴더 구조 없이 
    "migration:run": "npm run typeorm  migration:run", //실행
    "migration:revert": "npm run typeorm migration:revert"  마이그레이션 되돌리기

4. 마이그레이션 생성
npm run migration:generate -- -n  마이그레이션 이름 
이 명령어를 사용해 마이그레이션을 생성한다 

5. 마이그레이션 실행
npm run migration:run 사용해 마이그레이션 실행한다 

### 다른 ORM
이럴때는 prisma가 너무 좋다 마이그레이션도 쉽게 할 수 있다
npx prisma db push를 사용해 수동으로 관리하지 않고도 스키마를 빠르게 동기화를 할 수 있다 
