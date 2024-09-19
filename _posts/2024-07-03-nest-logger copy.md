---
layout: post
read_time: true
show_date: true
title: " nest에서 로깅해보기"
date: 2024-07-03 15:00:20 -0600
description: " 내장로거부터 원스턴 까지"
image: https://miro.medium.com/v2/resize:fit:820/0*xqTsRQ3MOnE1j2D9.png
tags: 
    - coding
    - diary
    - nest
author: soi

toc: no # leave empty or erase for no TOC
---
### 로깅이란?
프로그램이 실행되는 동안 중간 결과나 오류를 기록하는 것을 의미하고 로그는 프로그램의 동작상황을 추적하고 분석하는데 유용하다

nest는 이미 내장 로거를 사용해 로그를 출력하고 있다 

### 내장로거 
@nestjs/common에 포함되어 있으며 각 서비스 마다 로깅 옵션을 지정할 수 있고 글로벌로 작성할 수 도 있다

- 로그레벨(하나만 설정하면 해당 레벨보다 낮은 레벨의 로그도 모두 함께 출력)
1. log: 일반로그 메세지 
2. warn: 경고 메시지
3. error: 에러 메시지
4. debug: 디버그 정보
5. verbose: 상세로그

로거의 타임스탭프를 재정의 할 수 있고 기본 로거를 재정의하거나 기본 로거를 확장해 커스텀 로거를 작성한다 

개발환경에서는 디버그 로그가 있는게 좋지만 배포환경에서는 디버그 로그가 남지 않도록 하는게 좋다
```javascript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { Logger } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 환경 변수에 따라 로그 레벨 설정
  const isProduction = process.env.NODE_ENV === 'production';

  // 배포 환경에서는 log, error, warn 레벨만 활성화
  app.useLogger(isProduction ? ['log', 'error', 'warn'] : ['log', 'error', 'warn', 'debug', 'verbose']);

  await app.listen(3000);
}
bootstrap();
```

### 커스텀 로거
로거서비스를 확장해 커스텀 로거를 만든다 
```javascript
import { Injectable, LoggerService } from '@nestjs/common';

@Injectable()
export class MyCustomLogger implements LoggerService {
  log(message: string) {
    // 일반 로그
    console.log(`[LOG] ${new Date().toISOString()}: ${message}`);
  }

  error(message: string, trace: string) {
    // 에러 로그
    console.error(`[ERROR] ${new Date().toISOString()}: ${message} \nTrace: ${trace}`);
  }

  warn(message: string) {
    // 경고 로그
    console.warn(`[WARN] ${new Date().toISOString()}: ${message}`);
  }

  debug(message: string) {
    // 디버그 로그
    console.debug(`[DEBUG] ${new Date().toISOString()}: ${message}`);
  }

  verbose(message: string) {
    // 상세 로그
    console.log(`[VERBOSE] ${new Date().toISOString()}: ${message}`);
  }
}
```
커스텀 로거 또한 useLogger로 전역으로 사용할 수 있다 

### winston 로거 
winston을 사용해 콘솔 출력뿐만이 아니라 파일에 저장할 수도 있고 중요한 로드만 별도로 저장할 수 있다 

```
npm install --save winston  winston-daily-rotate-file
```

 WinstonModule 을 appModule에 import해 사용한다 

```javascript
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import * as path from 'path';
import * as DailyRotateFile from 'winston-daily-rotate-file';

@Module({
  imports: [
    WinstonModule.forRoot({
      transports: [
        // 콘솔에 로그 출력 (환경에 따라 다른 로그 레벨 적용)
        new winston.transports.Console({
          level: process.env.NODE_ENV === 'production' ? 'warn' : 'debug', //배포면 warn 레벨로(warn, log만), 개발이면 디버그 레벨로
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.colorize(),  // 콘솔에서 컬러로 로그 표시
            winston.format.printf(({ timestamp, level, message }) => {
              return `${timestamp} [${level}]: ${message}`;
            })
          ),
        }),

        // 날짜별로 모든 로그를 저장하는 파일 (Daily Rotate File)
        new DailyRotateFile({
          filename: path.join(__dirname, 'logs', 'all-%DATE%.log'), // 날짜별로 로그 파일 생성
          datePattern: 'YYYY-MM-DD',
          zippedArchive: true, // 로그 파일을 압축하여 보관
          maxSize: '20m', // 최대 20MB 파일 크기
          maxFiles: '14d', // 14일치 로그만 유지
          level: process.env.NODE_ENV === 'production' ? 'info' : 'debug', // 배포 환경은 info 이상, 개발 환경은 debug 이상
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.json() // JSON 형식으로 저장
          ),
        }),

        // 날짜별로 에러 로그만 저장하는 파일 (Daily Rotate File)
        new DailyRotateFile({
          filename: path.join(__dirname, 'logs', 'error-%DATE%.log'), // 날짜별로 에러 로그 파일 생성
          datePattern: 'YYYY-MM-DD',
          zippedArchive: true,
          maxSize: '20m',
          maxFiles: '30d', // 30일치 에러 로그만 유지
          level: 'error', // 에러 로그만 저장
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.json() // JSON 형식으로 저장
          ),
        }),
      ],
    }),
  ],
})
export class AppModule {}
```
