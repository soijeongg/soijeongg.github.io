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
controller: 사용할 컨트롤러
provider: 사용할 종속성들이 들어간다 
이곳에 추가해야한 nest가 인지하고 적용을 할 수 있다 
기능별로 폴더로 나누고 그 폴더별 모듈에 그 폴더의 종속성을 넣어주고 나온 폴더모듈을 appModule에 넣어준다 
(nest g mo name, nest g co name, nest g s name)

### 컨트롤러 
컨트롤러는 사용자의 요청을 받고 요청에 따라 서비스에 넘겨주고 마지막에 나온 return 값을 반환한다 
controller 어노테이션을 사용해 알려주고 생성자를 통해 서비스객체를 주입받는다 
``` typeScript
import { Controller, Body, Param, Get, Post, Put, Delete } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}
  @Get()
  async findAllController() {
    return this.userService.getAllService();
  }
}
```
지금보면 생성자로 유저서비스객체를 받아 사용하고 있다 

### 서비스 
비즈니스 로직을 처리하고 컨트롤러에 값을 반환한다 
찾아보니까 레포지토리 만들어도 되고 여기서 엔티티를 레포지토리 처럼 만들어서 사용한다 
하지만 복잡한 쿼리(여러 엔티티 조인, 서브쿼리)-> 특정 조건 기반으로 여러 테이블에서 데이터 조회해야 하는경우, 
재사용가능한 로직, 기본적인 crud가 아닌 경우에는 커스텀 레포지토리가 필요하다

```typeScript
import { Injectable,HttpException} from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entities';
//서비스는 모든 비즈니스 로직을 처리한다  -> 레포지토리는 오직 db접근만
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) private userRepository: Repository<User>,
  ) {}
  async getAllService() {
    return await this.userRepository.find();
  }
}
```
Injectable로 주입받고 생성자를 사용해 이 객체가 호출되었을때 엔티티를 이용해 레포지토리를 만든다 
그 후 crud를 진행한다(save, find, update, delete)

### DTO
dto는 데이터 전송객체로 계층간의 데이터 젆송에 사용된다 
데이터구조를 정의해 명확하게 개발이 가능하고 class-validator를 통해 유효성검사를 할 수 있다 





