---
layout: post
read_time: true
show_date: true
title: "타입스크립트+typeORM+express(1)"
date: 2024-06-16 15:00:20 -0600
description: "기본 구조 및 "
image: https://i.pinimg.com/736x/33/62/6a/33626ae9adfa3f30190d8734032315b2.jpg
tags: 
    - coding
    - diary
author: soi

toc: no # leave empty or erase for no TOC
---
책이나 웹소설, 전자책등을 보면서 인상깊었던 구절을 기록한다 

### 프로젝트 선정이유 
내가 활자중독이라서....
책이 없으면 웹서핑하고 볼게 없으면 지식백과라도 보는 사람...

### 구성
javascript+typescript+typeorm+RDS(postergalsql)
passport+jwt
gictaction+docker+jest+eslint+awsEC2
완성 후에 react를 사용해 간단한 페이지를 붙이거나 swift or react native를 사용해 덧붙일 계획이다 

- 기본 구성

node.js에서 하는것과 달리 구조가 달라서 조금 어려웠다 

src/
├── controllers/
│ └── userController.ts
├── entities/
│ └── user.ts
├── repositories/
│ └── userRepository.ts
├── services/
│ └── userService.ts
├── interfaces/
│ ├── iUserService.ts
│ ├── iUserRepository.ts
│ └── iUser.ts
| ├── router
| ├── index.ts
│ └── user
│ └── userRouter.ts
|
├── app.ts
├── data-source.ts
└── index.ts

#### 각 폴더들이 하는 역할

- 컨트롤러 레파지토리 서비스
원래 mvc의 3계층

- 엔티티
데이터테이블과 매핑되는 엔티티 클래스를 저장하는 디렉토리 
typeorm을 사용할때 각 typescript클래스로 정의되고 이 클래스 저장

- 인터페이스 
타입스크립트에서 사용하는 인터페이스를 저장하는 디렉토리 
인터페이스는 갹체의 타입을 정의하기 위해 사용된다
데이터페이스 엔티티와 매핑되는 인터페이스 라던가 , 서비스 인터페이스, 요청인터페이스등을 정의 한다 
실제 컨틀롤나 서비스에서 가져와 사용한다 

- data-source.ts
TypeORM을 사용하여 데이터베이스 연결을 설정
 데이터베이스와의 연결을 초기화하고 구성하는 데 사용

 - rds와 typeORM 연결하기
 dataSource.ts 파일에 rds의 정보를 작성한다 
```typescript
import 'reflect-metadata';
import { DataSource } from 'typeorm';
import { User } from './entities/user';
import dotenv from 'dotenv';

dotenv.config();

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: process.env.TYPEORM_HOST, //엔드포인트
  port: Number(process.env.TYPEORM_PORT),//포트 postgres는 5432
  username: process.env.TYPEORM_USERNAME, 
  password: process.env.TYPEORM_PASSWORD,
  database: process.env.TYPEORM_DATABASE,//만든 이름
  synchronize: process.env.TYPEORM_SYNCHRONIZE === 'true', //애리케이션을 시작할 때 엔티티와 데이터베이스 테이블을 자동으로 동기화
  "ssl": {
    "rejectUnauthorized": false
  }, //15버전부터 인증서 뭐가 바뀌었다 해서 추가 
  logging: process.env.TYPEORM_LOGGING === 'true',
  entities: [User], //매핑할 엔티티
  migrations: [],
  subscribers: [],
});

AppDataSource.initialize()
  .then(() => {
    console.log('Data Source has been initialized!');
  })
  .catch((err) => {
    console.error('Error during Data Source initialization:', err);
  });
```
### 엔티티 파일
```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn() //자동증가
  id!: number;

  @Column({ unique: true }) 
  email!: string;

  @Column()
  password!:string;

  @Column()
  name!: string;

}
```
뒤에 !가 붙은 이유는  2.7.2에부터 도입된 strict class checking으로 인한 오류때문이다 
[해당 오류 해결방법](https://github.com/songyouhyun/External-Brain/blob/master/Language/TypeScript/typescript_error.md)

### app.ts와 index.ts
원래 app.js에 넣어놨던걸 둘로 나눴다 
처음에 npm init으로 만들때 index.js를 main으로 잡아 따로 나눠놨다 
```typescript
//app.ts
import express, { Request, Response } from 'express'; //명시적으로 지정하기 위해 사용
import passport from 'passport';
import router from './router';
import bodyParser from 'body-parser';
import dotenv from 'dotenv';
import cors from 'cors';
import passportConfig from './utils/passportConfig'
import cookieParser from "cookie-parser"
import morgan  from "morgan"


dotenv.config();
const app = express();
app.use(cors());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(express.json());
app.use(cookieParser())
app.use(morgan("dev"))

app.use(passport.initialize());
passportConfig(passport);

app.get('/', async (req: Request, res: Response) => {
    res.send('<h1>시작하는 지점</h1>');
  });
  
app.use("api",router)

export default app;
```
```typescript
//index.ts
import  express from "express"
import app from "./app"
import dotenv from 'dotenv';
import { AppDataSource } from './dataSource';

dotenv.config();
const PORT = process.env.PORT || 3000;
AppDataSource.initialize().then(() => {
    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
  }).catch(error => {
    console.error('Error during Data Source initialization:', error);
  });
```

인덱스에 데이터 소스를 적용한다 
그 후 tsc하면 컴파일이 되고 node dist/index.js로 실행시킨다 
![](https://velog.velcdn.com/images/soijeongg/post/b2cee74d-a886-4b76-a3b9-42382d2ca755/image.png)

#### postgresql
postgresql을 설치한 후  옆 사이드 바에 있는 아이콘을 클릭한 후 +버튼을 누른다 

이 순서대로 입력한다 
1.RDS 엔드포인트 
2. 유저이름
3. 비밀번호 
4. 포트
5.use Secure Connection 선택
6.show all
7. 엔터 누르기

### 라우터 만들기 
라우터에서 각 요청을 컨트롤러로 보내준다 
이때 그냥 node.js에서 하는것처럼 express를 import하고 서비스계층과 컨트롤러계층을 주입한다 

이때 컨트롤러 계층을 그냥 만들것이 아니라 먼저  객체의 구조를 정의하는 인터페이스를 만든다 
인터페이스는 객체가 가져야 하는 속성과 메서드를 명시해 안전성을 보장하고 오류를 줄여준다 
컨트롤러 인터페이스를 만들어서 만들 메서드를 정의한 다음에 이다음에 컨트롤러 계층을 만든다 

### 컨트롤러 만들기 
인터페이스에  만들어놓은 정의된 메서드들을 보고 만든다 
그렇게 하기 위해 인터페이스를 import하고 생성자에서 주입해준다 
```typescript
import { Request, Response,NextFunction } from 'express';
import { IuserController } from '../interfaces/user/iuserCotroller';
import { UserService } from '../service/UserService';

export class UserController implements IuserController {
  private userService: UserService;

  constructor(userService: UserService) {
    this.userService = new UserService();
    this.signupController = this.signupController.bind(this);
    this.deleteUsercontroller = this.deleteUsercontroller.bind(this);
  }
```
implements를 사용해 이 클래스가 특정 인터페이스를 구현한다는걸 알려준다 
이 컨트롤러에서 사용되는 서비스가 어떤 서비스인지 생성자를 주입한다 
그후 CRUD에 해당하는 컨트롤러를 인터페이스를 참고해 만든다 
만든 후 서비스 계층으로 가 똑같이 만든 인터페이스를 보고 서비스 계층을 만든다 


