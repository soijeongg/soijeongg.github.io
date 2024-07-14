---
layout: post
read_time: true
show_date: true
title: "nest.js typeorm 연결 VS typescript+express의 typeorm 연결 "
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
기존의 타입스크립트를 사용할때 보다 nest를 사용하면 nest의 모듈들을 사용하면 좀 더 쉽게 연결을 할 수 있다.

### 기존 방법
기존의 바닐라 타입스크립트에서는 모듈이 없기 때문에 직접 DataSource를 import해 설정해야했었다.
```typeScript
export const AppDataSource = new DataSource({
  type: 'postgres',
  host: process.env.TYPEORM_HOST,
  port: Number(process.env.TYPEORM_PORT),
  username: process.env.TYPEORM_USERNAME,
  password: process.env.TYPEORM_PASSWORD,
  database: process.env.TYPEORM_DATABASE,
  synchronize: process.env.TYPEORM_SYNCHRONIZE === 'true',
  ssl: {
    rejectUnauthorized: false,
  },
  logging: process.env.TYPEORM_LOGGING === 'true',
  entities: [
    User,
    Posts,
    Comment,
    Categories,
    subCategories,
    AccessToken,
    RefreshToken,
    BlacklistedToken,
  ],
  migrations: ['src/database/migrations/**/*.ts'],
  subscribers: [],
});
```
이렇게 만들어진 데이터소스객체를 메인인 index.ts에서 초기화를 하고 레포지토리 폴더에서 레포지토리 객체를 데이터소스에서 getRepository 명령어를 사용해 가져와 사용한다.

### nest.js에서의 변화
nest.js에서는 모듈이 추가되어 이 모듈에 데이터베이스 연결정보를 넣으면 된다. 
datasource파일을 만들어 뺴서 정리한다.
```typeScript
import { TypeOrmModuleAsyncOptions } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { User } from './user/entities/user.entities';

ConfigModule.forRoot(); // .env 파일에서 환경 변수를 로드

const configService = new ConfigService();

//TypeOrmModuleAsyncOptions typeORM을 비동기적으로 사용 ->모듈 구성시 동젃으로 설정값 생성해 모듈에 주입
export const dataSourceOptions: TypeOrmModuleAsyncOptions = {
  imports: [ConfigModule], //ConfigModule를 import해서 환경변수 사용할수있게됨(로드)
  useFactory: async () => ({
    //설정객체를 동적으로 생성 configService를 주입받음
    type: 'postgres',
    host: configService.get<string>('TYPEORM_HOST'),
    port: configService.get<number>('TYPEORM_PORT'),
    username: configService.get<string>('TYPEORM_USERNAME'),
    password: configService.get<string>('TYPEORM_PASSWORD'),
    database: configService.get<string>('TYPEORM_DATABASE'),
    synchronize: configService.get<boolean>('TYPEORM_SYNCHRONIZE'),
    entities: [User],
  }),
  inject: [ConfigService], //환경변수에서 설정값 가져옴
};
```
**만약 모듈의 생성시점에 ConfigModule를 사용해 환경변수를 써야한다면**
nest에서 제공하는 config패키지는 nest에서 dotenv를 활용할수 있게 해준다.
ConfigModule를 직접 만들어도 되고 이렇게 기존의 것을 사용해도 된다.
ConfigModule은 정적이기 떄문에 모듈의 생성지점에서는 환경변수만을 로드 할 수 있다.
하지만 useFactory를 사용해 동적인 값을 받아 올 수 있고 모듈을 만들때 굳이 env를 안쓰더라도 동적으로 생성해 사용할 수 있다.

- ConfigService와 ConfigModule의 차이
ConfigModule은 환경 변수와 설정을 로드하고 관리하는 모듈입, .env파일을 읽는다.
ConfigService는 ConfigModule에서 로드한 환경변수와 설정값을 사용할수 있게 한다.