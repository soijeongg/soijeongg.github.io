---
layout: post
read_time: true
show_date: true
title: "nest.js의 구성요소 살펴보기 "
date: 2024-07-10 15:00:20 -0600
description: "nest의 모듈, 컨트롤러, 서비스, DI"
image: https://miro.medium.com/v2/resize:fit:820/0*xqTsRQ3MOnE1j2D9.png
tags: 
    - coding
    - diary
    - nest
author: soi

toc: no # leave empty or erase for no TOC
---
### 모듈
nest.js는 모듈 기반 아키텍쳐로 구성되어 있다 
모듈은 모듈 데코레이터로 어노테이션
``` typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
imports: 모듈이 정의하는 컨트롤러의 배열, 클라이언트의 요청을 처리하고 적절한 응답반환, 
         다른 모듈을 불러오고 설정하는 역할을 한다 
         또한 ORM연결 정의를 한다 
controller