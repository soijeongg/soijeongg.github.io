---
layout: post
read_time: true
show_date: true
title: "typeORM연결방법 및 레이어드 패턴에 interface와 DTO"
date: 2024-07-02 15:01:20 -0600
description: "typeORM연결방법 및 레이어드 패턴에 interface와 DTO" 
image: https://i.pinimg.com/564x/e5/dc/05/e5dc0598585f65e406f8b476e22da6cc.jpg
tags: 
    - coding
    - cs
author: soi

toc: no # leave empty or erase for no TOC
---

먼저 타입스크립트를 설치한 다음에 시작지점인 app과 index.ts를 만든다 
그 후 기초 설정을 한 다음에 인터체이스를 만든다 

### 인터페이스란
객체지향 프로그래밍에서의 개념으로 클래스나 객체가 구현해야하는 메서드의 속성의 집합을 정의 
어떤 메서드가 포함되어야 한다는 명세만 있는것
코드의 일관성을 유지하고 표준화된 방식으로 객체를 다룰 수 있게 하고 다형성을 구현하는데 도움 된다 
인테페이스를 사용하게 되면 인터페이스에 의존하게 되기 때문에 실제 구현을 쉽게 교체할 수 있어 유지보수와 확장성이 좋다 

-> 정리하면 인터페이스는 설계도, 청사진

이런 인터페이스를 컨트롤러, 서비스, 레포지토리, 엔티티를 정의한다
컨트롤러, 서비스, 레포지토리는 각 계층의 구조를 설계하고 리퀘스트는 어떤게 들어올지 미리 정의한다 
엔티티 인터페이스는 엔티티의 구조를 정의하고 타입 검사를 한다  

컨트롤러 인터페이스는 req, res, next를 인자로 받고 Response를 반환한다 
서비스 인터페이스는 컨트롤러에서 들어오는 DTO를 받아  유효성 검사등 비즈니스 로직을 실행한다 
그리고 다시 레포지토리계층으로 보낼때 DTO로 묶어 보낸다 

```typescript
//엔티티 인터페이스 
export interface IUser {
  userId: number;
  email: string;
  password: string;
  name: string;
}
```
이렇게 정의한 후 컨트롤러 인터페이스를 정의한다 
```typescript
export interface IuserController {
  signupController(
    req: Request,
    res: Response,
    next: NextFunction,
  ): Promise<Response | void>;
}
```
그 후 서비스 계층의 인터페이스를 정의한다  서비스계층은 컨트롤러에서 dto로 묶어서 오는 것을 받는다 
``` typescript
export interface IuserService {
  createUserService(user: createUserDTO): Promise<IUser>;
}
```
이렇게 넘어온다고 가정하고 create하고 save한것을 반환한다는 것으로 설정해서  Iuser의 형식을 반환한다고 설게했다 

```typescript
export interface IuserRespository {
  createUser(user: createUserDTO): Promise<IUser>;
}
```
레포지토리는 넘어오는 DTo를 받아 처리한다 

컨트롤러 -> 서비스: DTO 사용
서비스 내부: DTO를 도메인 모델이나 비즈니스 로직에 맞는 형태로 변환
서비스 -> 리포지토리: 필요에 따라 다른 DTO로 변환하여 전달

### typeORM을 datasource로 연결
타입스크립트 최신버전(3이상)에는 DataSource클래스를 통해  연결을 설정하는 것이 기본
이전 버전에서는 createConnection를 사용했지만 datasource를 사용하는 것이 권장

dataSource 를 사용해서 연결한다
```typescript
 import 'reflect-metadata'; //타입스크립트에서 데코레이터 기능을 사용하기 위해 필요한 메타데이터 api라이브러리
import { DataSource } from 'typeorm'; //데이터소스클래스, 데이터베이스 연결 및 설정관리
import {User} from './entities'; //엔티티 클래스 데이터베이스 테이블을 만듬
import * as dotenv from 'dotenv';
import path from 'path';

dotenv.config({ path: path.resolve(__dirname, '../.env') });

export const AppDataSource = new DataSource({
  type: 'postgres', //데이터베이스 종류
  host: process.env.TYPEORM_HOST, //호스트 로컬은 127.0.0.1이지만 RDS는 엔드포인트
  port: Number(process.env.TYPEORM_PORT), //포트 postgres는 기본 5432 mysql 3306
  username: process.env.TYPEORM_USERNAME,
  password: process.env.TYPEORM_PASSWORD,
  database: process.env.TYPEORM_DATABASE, //데이터베이스 이름, 로컹은 create로 만들고 RDS는 vscode 연결하고 데이터베이스를 만든다 
  synchronize: process.env.TYPEORM_SYNCHRONIZE === 'true', //시작시 자동으로 스키마 동기화여부
  ssl: {
    rejectUnauthorized: false,
  }, // aws 사용시 no pg_hba.conf entry 에러 연결을 강제 설정함 
  logging: process.env.TYPEORM_LOGGING === 'true', //로깅 설정
  entities: [User], //사용할 엔티티 파일 지정
  migrations: ['src/database/migrations/**/*.ts'], //마이그레이션파일 지정 
  subscribers: [],//이벤트 리스너와 구독자 위치 지정 엔티티 이벤트동안 로직 실행
});
```
postgresSQL 연결할떄 no pg_hba.conf entry에러 나올경우 [참고한 블로그](https://velog.io/@bshunter/AWS-ec2-rds-no-pghba.conf-entry-%EC%97%90%EB%9F%AC)
이후 초기화를 위해 app.ts에 AppDataSource.initialize()를 한다 

### 엔티티 설정
```typescript
mport { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';
import { Comment, Posts, AccessToken, RefreshToken } from './index';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  userId!: number;

  @Column({ unique: true })
  email!: string;

  @Column()
  password!: string;

  @Column()
  name!: string;

  @OneToMany(() => Comment, (Comment) => Comment.user)
  comments!: Comment[];

  @OneToMany(() => Posts, (Posts) => Posts.User)
  Posts!: Posts[];

  @OneToMany(() => AccessToken, (AccessToken) => AccessToken.user)
  accessTokens!: AccessToken[];

  @OneToMany(() => RefreshToken, (RefreshToken) => RefreshToken.user)
  RefeshTkoens!: RefreshToken[];
}
//한개의  유저는 여러개의 포스트를 작성할 수 있고 여러게의 포스트는 한명의 유저를 가질 수 있다
//한개의 유저는 여러개의 댓글을 작성할 수 있고 여러 댓글을 한 유저를 가진다
```
PrimaryGeneratedColumn 기본키인데 자동증가
 @OneToMany(() => Posts, (Posts) => Posts.User)
 첫번째 매개변수 () => Posts는 관계를 설정할 대상 엔티티 클래스 반환 순환참조 방지 위해서 직접 참조가 아니라 함수로 감싸서 전달한다
 두번째 매개변수 (posts) => posts.user는 함수로 첫번째 매개변수로 지정된 엔티티의 인스턴스를 받아 관게를 표현하는 프로퍼티에 반환한다 
